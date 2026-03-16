### 数据库配置文件结构

```
fastapi-project/
├── app/
│   ├── __init__.py
│   ├── main.py           # 主应用
│   ├── database.py       # 数据库配置
│   ├── models.py         # ORM 模型
│   ├── schemas.py        # Pydantic 模型
│   ├── crud.py           # 数据库操作
│   ├── dependencies.py   # 依赖函数
│   ├── config.py         # 配置文件
│   ├── routers/          # 路由模块
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── posts.py
│   │   └── auth.py
│   └── utils/            # 工具函数
│       ├── __init__.py
│       ├── security.py
│       └── email.py
├── tests/                # 测试文件
├── alembic/              # 数据库迁移
├── .env                  # 环境变量
├── requirements.txt      # 依赖
└── README.md
```

### database.py - 数据库配置

```
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.ext.asyncio import async_sessionmaker
from sqlalchemy.orm import declarative_base
import os

# 环境变量配置
DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql+asyncpg://user:password@localhost/mydb"
)

# 异步引擎
engine = create_async_engine(
    DATABASE_URL,
    echo=True,
    future=True,
    pool_pre_ping=True,  # 连接前检查
    pool_size=10,
    max_overflow=20,
    pool_recycle=3600,  # 1小时回收连接
)

# 会话工厂
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)

# 基类
Base = declarative_base()

# 依赖函数
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

# 创建所有表
async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

# 删除所有表
async def drop_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
```

### models.py - ORM 模型定义

```
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Text
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    username = Column(String(50), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 关系
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    profile = relationship("UserProfile", back_populates="user", uselist=False)

class UserProfile(Base):
    __tablename__ = "user_profiles"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True)
    bio = Column(Text, nullable=True)
    avatar_url = Column(String(500), nullable=True)
    
    user = relationship("User", back_populates="profile")

class Post(Base):
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    author_id = Column(Integer, ForeignKey("users.id"))
    published = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    author = relationship("User", back_populates="posts")
    tags = relationship("Tag", secondary="post_tags", back_populates="posts")

class Tag(Base):
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)
    
    posts = relationship("Post", secondary="post_tags", back_populates="tags")

# 多对多关联表
class PostTag(Base):
    __tablename__ = "post_tags"
    
    post_id = Column(Integer, ForeignKey("posts.id"), primary_key=True)
    tag_id = Column(Integer, ForeignKey("tags.id"), primary_key=True)
```

### schemas.py - Pydantic 模型

```
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional, List

# 用户 Schema
class UserBase(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = None
    password: Optional[str] = None

class UserInDB(UserBase):
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime
    
    class Config:
        from_attributes = True  # 允许从 ORM 模型创建

class User(UserInDB):
    pass

# 文章 Schema
class PostBase(BaseModel):
    title: str = Field(..., max_length=200)
    content: str

class PostCreate(PostBase):
    pass

class PostUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None
    published: Optional[bool] = None

class Post(PostBase):
    id: int
    author_id: int
    published: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

class PostWithAuthor(Post):
    author: User
```

### crud.py - 数据库 CRUD 操作

```
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update, delete
from sqlalchemy.orm import selectinload
from app.models import User, Post
from app.schemas import UserCreate, PostCreate
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# ========== 用户 CRUD ==========

async def get_user(db: AsyncSession, user_id: int):
    """根据 ID 获取用户"""
    result = await db.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()

async def get_user_by_email(db: AsyncSession, email: str):
    """根据邮箱获取用户"""
    result = await db.execute(
        select(User).where(User.email == email)
    )
    return result.scalar_one_or_none()

async def get_users(
    db: AsyncSession,
    skip: int = 0,
    limit: int = 100
):
    """获取用户列表"""
    result = await db.execute(
        select(User).offset(skip).limit(limit)
    )
    return result.scalars().all()

async def create_user(db: AsyncSession, user: UserCreate):
    """创建用户"""
    hashed_password = pwd_context.hash(user.password)
    db_user = User(
        email=user.email,
        username=user.username,
        hashed_password=hashed_password
    )
    db.add(db_user)
    await db.commit()
    await db.refresh(db_user)
    return db_user

async def update_user(db: AsyncSession, user_id: int, updates: dict):
    """更新用户"""
    if "password" in updates:
        updates["hashed_password"] = pwd_context.hash(updates.pop("password"))
    
    await db.execute(
        update(User)
        .where(User.id == user_id)
        .values(**updates)
    )
    await db.commit()
    return await get_user(db, user_id)

async def delete_user(db: AsyncSession, user_id: int):
    """删除用户"""
    await db.execute(
        delete(User).where(User.id == user_id)
    )
    await db.commit()

# ========== 文章 CRUD ==========

async def get_post(db: AsyncSession, post_id: int):
    """获取文章（包含作者信息）"""
    result = await db.execute(
        select(Post)
        .options(selectinload(Post.author))  # 预加载关联数据
        .where(Post.id == post_id)
    )
    return result.scalar_one_or_none()

async def get_posts(
    db: AsyncSession,
    skip: int = 0,
    limit: int = 100,
    published_only: bool = True
):
    """获取文章列表"""
    query = select(Post).options(selectinload(Post.author))
    
    if published_only:
        query = query.where(Post.published == True)
    
    result = await db.execute(
        query.offset(skip).limit(limit).order_by(Post.created_at.desc())
    )
    return result.scalars().all()

async def create_post(
    db: AsyncSession,
    post: PostCreate,
    author_id: int
):
    """创建文章"""
    db_post = Post(**post.dict(), author_id=author_id)
    db.add(db_post)
    await db.commit()
    await db.refresh(db_post)
    return db_post

async def update_post(db: AsyncSession, post_id: int, updates: dict):
    """更新文章"""
    await db.execute(
        update(Post)
        .where(Post.id == post_id)
        .values(**updates)
    )
    await db.commit()
    return await get_post(db, post_id)

async def delete_post(db: AsyncSession, post_id: int):
    """删除文章"""
    await db.execute(
        delete(Post).where(Post.id == post_id)
    )
    await db.commit()
```
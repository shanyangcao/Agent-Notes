### 一、为什么需要密码加密

**安全风险**：

- 明文存储密码极易泄露
- 数据库被攻击时用户信息全部暴露
- 彩虹表攻击可快速破解简单密码

**加密原则**：

- 单向加密（不可逆）
- 加盐（Salt）防止彩虹表攻击
- 慢速算法（增加暴力破解成本）

### 二、Passlib 配置

```
pip install "passlib[bcrypt]"
```

```
from passlib.context import CryptContext

# 创建密码上下文
pwd_context = CryptContext(
    schemes=["bcrypt"],  # 使用 bcrypt 算法
    deprecated="auto"    # 自动标记过时的哈希
)

# 也可以配置多个方案
pwd_context = CryptContext(
    schemes=["bcrypt", "argon2"],
    deprecated="auto",
    bcrypt__rounds=12,  # bcrypt 迭代次数
)
```

### 三、密码加密与验证

```
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# 加密密码
def hash_password(password: str) -> str:
    """将明文密码加密"""
    return pwd_context.hash(password)

# 验证密码
def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码是否正确"""
    return pwd_context.verify(plain_password, hashed_password)

# 示例
password = "my_secret_password123"
hashed = hash_password(password)
print(hashed)  
# $2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW

# 验证
is_valid = verify_password(password, hashed)
print(is_valid)  # True

is_valid = verify_password("wrong_password", hashed)
print(is_valid)  # False
```

### 四、在用户注册中使用

```
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.schemas import UserCreate, User
from app import crud

router = APIRouter(prefix="/auth", tags=["authentication"])

@router.post("/register", response_model=User, status_code=status.HTTP_201_CREATED)
async def register(
    user: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    """用户注册"""
    # 检查邮箱是否已存在
    existing_user = await crud.get_user_by_email(db, user.email)
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # 创建用户（密码会在 crud.create_user 中加密）
    new_user = await crud.create_user(db, user)
    return new_user
```

### 五、用户登录验证

```
from datetime import datetime, timedelta
from jose import JWTError, jwt
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

# JWT 配置
SECRET_KEY = "your-secret-key-keep-it-secret"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/login")

def create_access_token(data: dict, expires_delta: timedelta = None):
    """创建 JWT token"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

@router.post("/login")
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db)
):
    """用户登录"""
    # 查询用户
    user = await crud.get_user_by_email(db, form_data.username)
    
    # 验证用户存在且密码正确
    if not user or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    # 创建 token
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.email},
        expires_delta=access_token_expires
    )
    
    return {
        "access_token": access_token,
        "token_type": "bearer"
    }

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
):
    """从 token 获取当前用户"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        email: str = payload.get("sub")
        if email is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = await crud.get_user_by_email(db, email)
    if user is None:
        raise credentials_exception
    
    return user

# 受保护的路由
@router.get("/me", response_model=User)
async def read_users_me(current_user = Depends(get_current_user)):
    """获取当前用户信息"""
    return current_user
```

### 六、密码重置流程

```
import secrets
from datetime import datetime, timedelta

# 生成重置令牌
def generate_reset_token():
    """生成密码重置令牌"""
    return secrets.token_urlsafe(32)

# 存储令牌（需要在数据库中添加字段）
class User(Base):
    # ... 其他字段
    reset_token = Column(String(100), nullable=True)
    reset_token_expires = Column(DateTime(timezone=True), nullable=True)

@router.post("/forgot-password")
async def forgot_password(
    email: str,
    db: AsyncSession = Depends(get_db)
):
    """请求密码重置"""
    user = await crud.get_user_by_email(db, email)
    if not user:
        # 安全起见，不透露用户是否存在
        return {"message": "If email exists, reset link will be sent"}
    
    # 生成令牌
    reset_token = generate_reset_token()
    expires = datetime.utcnow() + timedelta(hours=1)
    
    # 更新用户
    await crud.update_user(db, user.id, {
        "reset_token": reset_token,
        "reset_token_expires": expires
    })
    
    # 发送邮件（此处省略）
    # send_reset_email(user.email, reset_token)
    
    return {"message": "Password reset email sent"}

@router.post("/reset-password")
async def reset_password(
    token: str,
    new_password: str,
    db: AsyncSession = Depends(get_db)
):
    """重置密码"""
    # 查找令牌对应的用户
    result = await db.execute(
        select(User).where(
            and_(
                User.reset_token == token,
                User.reset_token_expires > datetime.utcnow()
            )
        )
    )
    user = result.scalar_one_or_none()
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid or expired reset token"
        )
    
    # 更新密码并清除令牌
    await crud.update_user(db, user.id, {
        "password": new_password,  # crud 会自动加密
        "reset_token": None,
        "reset_token_expires": None
    })
    
    return {"message": "Password reset successful"}
```
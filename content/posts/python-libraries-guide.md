+++
title = "Python 常用第三方库指南"
date = "2026-03-16T22:20:00+08:00"

[taxonomies]
tags = ["python", "第三方库", "编程", "生态"]
categories = ["编程"]

[extra]
summary = "Python 拥有丰富的第三方库生态，本文介绍 Web 开发、数据科学、机器学习、自动化等常用库，帮助你快速构建 Python 应用。"
author = "博主"
+++

Python 以其简洁的语法和强大的生态系统而闻名。除了标准库外，Python 拥有数以万计的第三方库，涵盖几乎所有编程领域。本文将介绍各类常用第三方库，帮助你更高效地开发 Python 应用。

## Web 开发

### Django

[Django](https://www.djangoproject.com/) 是功能齐全的 Web 框架，遵循"包含电池"的理念。

```python
# models.py
from django.db import models
from django.contrib.auth.models import User

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title

# views.py
from django.shortcuts import render, get_object_or_404
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from .models import Article

@require_http_methods(["GET", "POST"])
def article_list(request):
    if request.method == 'POST':
        # 创建文章
        article = Article.objects.create(
            title=request.POST.get('title'),
            content=request.POST.get('content'),
            author=request.user
        )
        return JsonResponse({'id': article.id, 'status': 'created'})
    
    # 获取文章列表
    articles = Article.objects.filter(published=True)
    return render(request, 'articles/list.html', {'articles': articles})

def article_detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    return render(request, 'articles/detail.html', {'article': article})

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('articles/', views.article_list, name='article_list'),
    path('articles/<int:pk>/', views.article_detail, name='article_detail'),
]

# Django REST Framework 序列化器
from rest_framework import serializers, viewsets

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'created_at']

class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```

### Flask

[Flask](https://flask.palletsprojects.com/) 是轻量级的 Web 框架，灵活且易于扩展。

```python
from flask import Flask, request, jsonify, render_template
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow
from flask_jwt_extended import JWTManager, jwt_required, create_access_token
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['JWT_SECRET_KEY'] = 'your-secret-key'

db = SQLAlchemy(app)
ma = Marshmallow(app)
jwt = JWTManager(app)

# 数据库模型
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    user = db.relationship('User', backref=db.backref('posts', lazy=True))

# 序列化 Schema
class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User

class PostSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Post
        include_fk = True

user_schema = UserSchema()
posts_schema = PostSchema(many=True)

# 路由
@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/users', methods=['GET', 'POST'])
def users():
    if request.method == 'POST':
        data = request.get_json()
        new_user = User(username=data['username'], email=data['email'])
        db.session.add(new_user)
        db.session.commit()
        return jsonify(user_schema.dump(new_user)), 201
    
    all_users = User.query.all()
    return jsonify(user_schema.dump(all_users, many=True))

@app.route('/api/posts', methods=['GET'])
@jwt_required()
def get_posts():
    posts = Post.query.all()
    return jsonify(posts_schema.dump(posts))

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username', None)
    access_token = create_access_token(identity=username)
    return jsonify(access_token=access_token)

# 错误处理
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)
```

### FastAPI

[FastAPI](https://fastapi.tiangolo.com/) 是现代、高性能的 Web 框架，基于 Python 类型提示。

```python
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel, EmailStr
from typing import List, Optional
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext
import uvicorn

app = FastAPI(title="My API", version="1.0.0")

# 安全设置
SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# Pydantic 模型
class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None
    disabled: Optional[bool] = None

class UserCreate(UserBase):
    password: str

class User(UserBase):
    id: int
    created_at: datetime
    
    class Config:
        from_attributes = True

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

class Token(BaseModel):
    access_token: str
    token_type: str

# 模拟数据库
fake_users_db = {}

# 依赖项
def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = fake_users_db.get(username)
    if user is None:
        raise credentials_exception
    return user

# 路由
@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = fake_users_db.get(form_data.username)
    if not user or not pwd_context.verify(form_data.password, user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
        )
    
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = jwt.encode(
        {"sub": user["username"], "exp": datetime.utcnow() + access_token_expires},
        SECRET_KEY,
        algorithm=ALGORITHM
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user

@app.post("/users/", response_model=User)
async def create_user(user: UserCreate):
    hashed_password = pwd_context.hash(user.password)
    user_dict = user.dict()
    user_dict["hashed_password"] = hashed_password
    del user_dict["password"]
    user_dict["id"] = len(fake_users_db) + 1
    user_dict["created_at"] = datetime.now()
    fake_users_db[user.username] = user_dict
    return user_dict

@app.get("/users/", response_model=List[User])
async def read_users(skip: int = 0, limit: int = 100):
    return list(fake_users_db.values())[skip : skip + limit]

@app.get("/users/{user_id}", response_model=User)
async def read_user(user_id: int):
    for user in fake_users_db.values():
        if user["id"] == user_id:
            return user
    raise HTTPException(status_code=404, detail="User not found")

@app.post("/items/")
async def create_item(item: Item, current_user: User = Depends(get_current_user)):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict

@app.get("/items/")
async def read_items(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

# 启动服务器
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 数据科学与分析

### NumPy

[NumPy](https://numpy.org/) 是 Python 科学计算的基础库，提供高性能的多维数组对象。

```python
import numpy as np

# 创建数组
arr1 = np.array([1, 2, 3, 4, 5])
arr2 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 特殊数组
zeros = np.zeros((3, 4))
ones = np.ones((2, 3, 4))
identity = np.eye(3)
arange = np.arange(0, 10, 2)
linspace = np.linspace(0, 1, 100)

# 数组属性
print(arr2.shape)      # (3, 3)
print(arr2.ndim)       # 2
print(arr2.size)       # 9
print(arr2.dtype)      # int64

# 数组操作
reshaped = arr2.reshape(1, 9)
transposed = arr2.T
flattened = arr2.flatten()

# 数学运算
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# 元素级运算
print(a + b)
print(a * b)           # 元素乘法
print(a / b)
print(np.sqrt(a))

# 矩阵运算
print(a @ b)           # 矩阵乘法
print(np.dot(a, b))    # 同上
print(np.linalg.inv(a))  # 逆矩阵
print(np.linalg.det(a))  # 行列式
print(np.linalg.eig(a))  # 特征值和特征向量

# 广播
print(a + 10)          # 每个元素加 10
print(a + np.array([10, 20]))  # 广播到每行

# 索引和切片
print(arr2[0, :])      # 第一行
print(arr2[:, 1])      # 第二列
print(arr2[1:3, 1:3])  # 子数组

# 条件筛选
print(arr2[arr2 > 5])  # 大于 5 的元素
print(np.where(arr2 > 5, arr2, 0))  # 条件赋值

# 统计函数
print(np.mean(arr2))
print(np.std(arr2))
print(np.sum(arr2, axis=0))  # 按列求和
print(np.max(arr2, axis=1))  # 每行最大值

# 随机数
np.random.seed(42)
random_arr = np.random.rand(3, 3)
normal_dist = np.random.normal(0, 1, 1000)
random_int = np.random.randint(1, 100, (3, 3))

# 文件 I/O
np.save('array.npy', arr2)
loaded = np.load('array.npy')
np.savetxt('array.csv', arr2, delimiter=',')
loaded_csv = np.loadtxt('array.csv', delimiter=',')
```

### Pandas

[Pandas](https://pandas.pydata.org/) 提供高性能、易用的数据结构和数据分析工具。

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# 创建 Series
s = pd.Series([1, 3, 5, np.nan, 6, 8])
print(s)

# 创建 DataFrame
df = pd.DataFrame({
    'A': 1.0,
    'B': pd.Timestamp('20230102'),
    'C': pd.Series(1, index=list(range(4)), dtype='float32'),
    'D': np.array([3] * 4, dtype='int32'),
    'E': pd.Categorical(["test", "train", "test", "train"]),
    'F': 'foo'
})

# 从字典创建
data = {
    'name': ['张三', '李四', '王五', '赵六'],
    'age': [25, 30, 35, 28],
    'city': ['北京', '上海', '广州', '深圳'],
    'salary': [5000, 8000, 12000, 15000]
}
df = pd.DataFrame(data)

# 查看数据
print(df.head())
print(df.tail(3))
print(df.info())
print(df.describe())
print(df.shape)

# 选择数据
print(df['name'])           # 选择列
print(df[['name', 'age']])  # 选择多列
print(df.iloc[0])           # 按位置选择
print(df.loc[0, 'name'])     # 按标签选择
print(df[df['age'] > 28])   # 条件筛选
print(df.query('age > 28 and salary > 10000'))

# 数据清洗
df['age'] = df['age'].fillna(df['age'].mean())  # 填充缺失值
df = df.dropna()  # 删除缺失值
df = df.drop_duplicates()  # 删除重复值

# 数据转换
df['age_group'] = pd.cut(df['age'], bins=[0, 30, 40, 100], labels=['青年', '中年', '老年'])
df['salary_k'] = df['salary'] / 1000
df['name_upper'] = df['name'].str.upper()

# 排序和分组
df_sorted = df.sort_values('salary', ascending=False)
grouped = df.groupby('city').agg({
    'salary': ['mean', 'max', 'min'],
    'age': 'mean'
})

# 合并数据
df1 = pd.DataFrame({'key': ['A', 'B', 'C'], 'value1': [1, 2, 3]})
df2 = pd.DataFrame({'key': ['A', 'B', 'D'], 'value2': [4, 5, 6]})

merged = pd.merge(df1, df2, on='key', how='inner')
concatenated = pd.concat([df1, df2], axis=0)

# 时间序列
dates = pd.date_range('20230101', periods=100, freq='D')
ts = pd.Series(np.random.randn(100), index=dates)
print(ts.resample('M').mean())  # 按月重采样
print(ts.rolling(window=7).mean())  # 移动平均

# 数据透视表
pivot = pd.pivot_table(df, values='salary', index='city', 
                       columns='age_group', aggfunc='mean')

# 文件 I/O
df.to_csv('data.csv', index=False, encoding='utf-8')
df.to_excel('data.xlsx', sheet_name='Sheet1')
df.to_json('data.json', orient='records', force_ascii=False)

df_csv = pd.read_csv('data.csv')
df_excel = pd.read_excel('data.xlsx')
df_json = pd.read_json('data.json')

# 从 SQL 读取
from sqlalchemy import create_engine
engine = create_engine('sqlite:///database.db')
df_sql = pd.read_sql('SELECT * FROM users', engine)
df.to_sql('users', engine, if_exists='replace', index=False)
```

### Matplotlib & Seaborn

[Matplotlib](https://matplotlib.org/) 是 Python 的绘图库，[Seaborn](https://seaborn.pydata.org/) 基于 Matplotlib 提供更美观的统计图形。

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd

# 设置样式
plt.style.use('seaborn-v0_8')
sns.set_palette("husl")

# 生成示例数据
np.random.seed(42)
x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.cos(x)

# 基础线图
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y1, label='sin(x)', linewidth=2)
ax.plot(x, y2, label='cos(x)', linewidth=2, linestyle='--')
ax.set_xlabel('X 轴', fontsize=12)
ax.set_ylabel('Y 轴', fontsize=12)
ax.set_title('三角函数', fontsize=14)
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('line_plot.png', dpi=300)
plt.show()

# 散点图
data = pd.DataFrame({
    'x': np.random.randn(100),
    'y': np.random.randn(100),
    'category': np.random.choice(['A', 'B', 'C'], 100)
})

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Matplotlib 散点图
axes[0].scatter(data['x'], data['y'], c=data['category'].astype('category').cat.codes, 
                cmap='viridis', alpha=0.6)
axes[0].set_title('Matplotlib 散点图')

# Seaborn 散点图
sns.scatterplot(data=data, x='x', y='y', hue='category', ax=axes[1], s=100)
axes[1].set_title('Seaborn 散点图')

plt.tight_layout()
plt.show()

# 柱状图
categories = ['产品A', '产品B', '产品C', '产品D']
values = [23, 45, 56, 78]

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 垂直柱状图
axes[0].bar(categories, values, color=['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4'])
axes[0].set_ylabel('销量')
axes[0].set_title('产品销量对比')

# 水平柱状图
axes[1].barh(categories, values, color=['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4'])
axes[1].set_xlabel('销量')
axes[1].set_title('产品销量对比（水平）')

plt.tight_layout()
plt.show()

# 直方图和密度图
data = np.random.normal(100, 15, 1000)

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].hist(data, bins=30, edgecolor='black', alpha=0.7)
axes[0].set_xlabel('值')
axes[0].set_ylabel('频数')
axes[0].set_title('直方图')

sns.histplot(data, kde=True, ax=axes[1])
axes[1].set_title('带密度曲线的直方图')

plt.tight_layout()
plt.show()

# 箱线图和热图
df = pd.DataFrame(np.random.randn(100, 4), columns=['A', 'B', 'C', 'D'])

fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 箱线图
df.boxplot(ax=axes[0, 0])
axes[0, 0].set_title('箱线图')

# Seaborn 箱线图
sns.boxplot(data=df, ax=axes[0, 1])
axes[0, 1].set_title('Seaborn 箱线图')

# 小提琴图
sns.violinplot(data=df, ax=axes[1, 0])
axes[1, 0].set_title('小提琴图')

# 热图
corr = df.corr()
sns.heatmap(corr, annot=True, cmap='coolwarm', center=0, ax=axes[1, 1])
axes[1, 1].set_title('相关性热图')

plt.tight_layout()
plt.show()

# 多子图
fig = plt.figure(figsize=(15, 10))

gs = fig.add_gridspec(3, 3, hspace=0.3, wspace=0.3)

ax1 = fig.add_subplot(gs[0, :])
ax1.plot(x, y1)
ax1.set_title('顶部大图')

ax2 = fig.add_subplot(gs[1, 0])
ax2.plot(x, y2)

ax3 = fig.add_subplot(gs[1, 1])
ax3.scatter(np.random.randn(50), np.random.randn(50))

ax4 = fig.add_subplot(gs[1, 2])
ax4.bar(['A', 'B', 'C'], [1, 2, 3])

ax5 = fig.add_subplot(gs[2, :2])
ax5.hist(np.random.randn(1000), bins=20)

ax6 = fig.add_subplot(gs[2, 2])
ax6.pie([30, 20, 50], labels=['A', 'B', 'C'], autopct='%1.1f%%')

plt.show()
```

## 机器学习

### Scikit-learn

[Scikit-learn](https://scikit-learn.org/) 是简单且高效的数据挖掘和数据分析工具。

```python
from sklearn.datasets import load_iris, load_boston, make_classification
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix
import numpy as np
import pandas as pd

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 数据分割
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 数据标准化
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 分类模型
models = {
    'Logistic Regression': LogisticRegression(max_iter=1000),
    'Decision Tree': DecisionTreeClassifier(random_state=42),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42),
    'SVM': SVC(kernel='rbf', random_state=42),
    'KNN': KNeighborsClassifier(n_neighbors=5)
}

# 训练和评估
for name, model in models.items():
    model.fit(X_train_scaled, y_train)
    y_pred = model.predict(X_test_scaled)
    accuracy = accuracy_score(y_test, y_pred)
    print(f"{name}: {accuracy:.4f}")

# 交叉验证
rf = RandomForestClassifier(n_estimators=100, random_state=42)
cv_scores = cross_val_score(rf, X, y, cv=5)
print(f"Cross-validation scores: {cv_scores}")
print(f"Mean CV score: {cv_scores.mean():.4f} (+/- {cv_scores.std() * 2:.4f})")

# 超参数调优
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 7, None],
    'min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)

grid_search.fit(X_train_scaled, y_train)
print(f"Best parameters: {grid_search.best_params_}")
print(f"Best score: {grid_search.best_score_:.4f}")

# 特征重要性
best_model = grid_search.best_estimator_
feature_importance = pd.DataFrame({
    'feature': iris.feature_names,
    'importance': best_model.feature_importances_
}).sort_values('importance', ascending=False)

print(feature_importance)

# 聚类
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
clusters = kmeans.fit_predict(X)

# PCA 降维
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
print(f"Total variance explained: {sum(pca.explained_variance_ratio_):.4f}")

# 回归示例
from sklearn.datasets import fetch_california_housing

housing = fetch_california_housing()
X_reg, y_reg = housing.data, housing.target

X_train_reg, X_test_reg, y_train_reg, y_test_reg = train_test_split(
    X_reg, y_reg, test_size=0.2, random_state=42
)

regressor = LinearRegression()
regressor.fit(X_train_reg, y_train_reg)
y_pred_reg = regressor.predict(X_test_reg)

from sklearn.metrics import mean_squared_error, r2_score
mse = mean_squared_error(y_test_reg, y_pred_reg)
r2 = r2_score(y_test_reg, y_pred_reg)

print(f"MSE: {mse:.4f}")
print(f"R² Score: {r2:.4f}")

# 模型保存和加载
import joblib

joblib.dump(best_model, 'model.pkl')
loaded_model = joblib.load('model.pkl')
```

### TensorFlow / Keras

[TensorFlow](https://www.tensorflow.org/) 是端到端的开源机器学习平台。

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, models, optimizers, callbacks
import numpy as np

# 检查版本和 GPU
print(f"TensorFlow version: {tf.__version__}")
print(f"GPU available: {tf.config.list_physical_devices('GPU')}")

# 加载数据
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()

# 数据预处理
x_train = x_train.astype('float32') / 255.0
x_test = x_test.astype('float32') / 255.0
x_train = x_train.reshape(-1, 28, 28, 1)
x_test = x_test.reshape(-1, 28, 28, 1)

y_train = keras.utils.to_categorical(y_train, 10)
y_test = keras.utils.to_categorical(y_test, 10)

# 构建 CNN 模型
model = models.Sequential([
    layers.Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.Flatten(),
    layers.Dense(64, activation='relu'),
    layers.Dropout(0.5),
    layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

model.summary()

# 训练模型
early_stopping = callbacks.EarlyStopping(
    monitor='val_loss',
    patience=3,
    restore_best_weights=True
)

reduce_lr = callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.2,
    patience=2,
    min_lr=0.0001
)

history = model.fit(
    x_train, y_train,
    batch_size=128,
    epochs=20,
    validation_split=0.1,
    callbacks=[early_stopping, reduce_lr],
    verbose=1
)

# 评估模型
test_loss, test_acc = model.evaluate(x_test, y_test, verbose=0)
print(f"Test accuracy: {test_acc:.4f}")

# 保存模型
model.save('mnist_model.h5')
loaded_model = keras.models.load_model('mnist_model.h5')

# 使用函数式 API 构建复杂模型
inputs = layers.Input(shape=(28, 28, 1))
x = layers.Conv2D(32, (3, 3), activation='relu')(inputs)
x = layers.MaxPooling2D((2, 2))(x)
x = layers.Conv2D(64, (3, 3), activation='relu')(x)
x = layers.MaxPooling2D((2, 2))(x)
x = layers.Flatten()(x)
x = layers.Dense(64, activation='relu')(x)
outputs = layers.Dense(10, activation='softmax')(x)

functional_model = models.Model(inputs=inputs, outputs=outputs)

# 自定义训练循环
@tf.function
def train_step(x, y, model, optimizer, loss_fn):
    with tf.GradientTape() as tape:
        predictions = model(x, training=True)
        loss = loss_fn(y, predictions)
    
    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    return loss

# 使用预训练模型
base_model = keras.applications.ResNet50(
    weights='imagenet',
    include_top=False,
    input_shape=(224, 224, 3)
)

base_model.trainable = False  # 冻结预训练层

inputs = layers.Input(shape=(224, 224, 3))
x = base_model(inputs, training=False)
x = layers.GlobalAveragePooling2D()(x)
x = layers.Dense(256, activation='relu')(x)
outputs = layers.Dense(10, activation='softmax')(x)

transfer_model = models.Model(inputs, outputs)
```

### PyTorch

[PyTorch](https://pytorch.org/) 是开源的机器学习框架，以动态计算图著称。

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F
from torch.utils.data import DataLoader, Dataset
from torchvision import datasets, transforms
import numpy as np

# 检查设备
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# 数据预处理
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5,), (0.5,))
])

# 加载 MNIST 数据集
train_dataset = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
test_dataset = datasets.MNIST(root='./data', train=False, download=True, transform=transform)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False)

# 定义神经网络
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(1, 32, 3, 1)
        self.conv2 = nn.Conv2d(32, 64, 3, 1)
        self.dropout1 = nn.Dropout(0.25)
        self.dropout2 = nn.Dropout(0.5)
        self.fc1 = nn.Linear(9216, 128)
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        x = self.conv1(x)
        x = F.relu(x)
        x = self.conv2(x)
        x = F.relu(x)
        x = F.max_pool2d(x, 2)
        x = self.dropout1(x)
        x = torch.flatten(x, 1)
        x = self.fc1(x)
        x = F.relu(x)
        x = self.dropout2(x)
        x = self.fc2(x)
        output = F.log_softmax(x, dim=1)
        return output

model = Net().to(device)
print(model)

# 定义损失函数和优化器
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=1, gamma=0.7)

# 训练函数
def train(model, device, train_loader, optimizer, epoch):
    model.train()
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)
        
        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()
        
        if batch_idx % 100 == 0:
            print(f'Train Epoch: {epoch} [{batch_idx * len(data)}/{len(train_loader.dataset)}'
                  f' ({100. * batch_idx / len(train_loader):.0f}%)]\tLoss: {loss.item():.6f}')

# 测试函数
def test(model, device, test_loader):
    model.eval()
    test_loss = 0
    correct = 0
    
    with torch.no_grad():
        for data, target in test_loader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            test_loss += criterion(output, target).item()
            pred = output.argmax(dim=1, keepdim=True)
            correct += pred.eq(target.view_as(pred)).sum().item()
    
    test_loss /= len(test_loader)
    accuracy = 100. * correct / len(test_loader.dataset)
    
    print(f'\nTest set: Average loss: {test_loss:.4f}, '
          f'Accuracy: {correct}/{len(test_loader.dataset)} ({accuracy:.2f}%)\n')

# 训练模型
for epoch in range(1, 11):
    train(model, device, train_loader, optimizer, epoch)
    test(model, device, test_loader)
    scheduler.step()

# 保存模型
torch.save(model.state_dict(), 'mnist_cnn.pt')

# 加载模型
model = Net().to(device)
model.load_state_dict(torch.load('mnist_cnn.pt'))

# 使用 DataLoader 自定义数据集
class CustomDataset(Dataset):
    def __init__(self, data, labels, transform=None):
        self.data = data
        self.labels = labels
        self.transform = transform

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        sample = self.data[idx]
        label = self.labels[idx]
        
        if self.transform:
            sample = self.transform(sample)
        
        return sample, label

# 使用预训练模型
import torchvision.models as models

resnet = models.resnet50(pretrained=True)

# 冻结层
for param in resnet.parameters():
    param.requires_grad = False

# 修改最后一层
num_ftrs = resnet.fc.in_features
resnet.fc = nn.Linear(num_ftrs, 10)

resnet = resnet.to(device)
```

## 自动化与脚本

### Requests

[Requests](https://requests.readthedocs.io/) 是简洁优雅的 HTTP 库。

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry
import json

# 基本 GET 请求
response = requests.get('https://api.github.com')
print(response.status_code)
print(response.headers['content-type'])
print(response.json())

# 带参数的 GET
params = {'q': 'python', 'page': 1}
response = requests.get('https://api.github.com/search/repositories', params=params)
print(response.url)

# POST 请求
data = {'key': 'value'}
response = requests.post('https://httpbin.org/post', data=data)

# JSON 数据
json_data = {'name': 'test', 'value': 123}
response = requests.post('https://httpbin.org/post', json=json_data)

# 自定义 Headers
headers = {
    'User-Agent': 'MyApp/1.0',
    'Authorization': 'Bearer token123'
}
response = requests.get('https://api.example.com', headers=headers)

# 表单数据
files = {'file': open('report.txt', 'rb')}
response = requests.post('https://httpbin.org/post', files=files)

# 超时设置
try:
    response = requests.get('https://api.example.com', timeout=5)
except requests.Timeout:
    print('Request timed out')

# 会话保持
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token'})

# 重试策略
retry_strategy = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
)
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)

# 流式下载
with requests.get('https://example.com/largefile.zip', stream=True) as r:
    r.raise_for_status()
    with open('largefile.zip', 'wb') as f:
        for chunk in r.iter_content(chunk_size=8192):
            f.write(chunk)

# 代理设置
proxies = {
    'http': 'http://10.10.1.10:3128',
    'https': 'http://10.10.1.10:1080',
}
response = requests.get('https://api.example.com', proxies=proxies)

# SSL 验证
response = requests.get('https://api.example.com', verify=False)  # 不推荐
response = requests.get('https://api.example.com', verify='/path/to/certfile')

# 响应处理
response = requests.get('https://api.github.com')
print(response.ok)           # 状态码 < 400
print(response.is_redirect)  # 是否重定向
print(response.links)        # 解析 Link 头
print(response.cookies)      # Cookie 信息

# 认证
from requests.auth import HTTPBasicAuth
response = requests.get('https://api.example.com', auth=HTTPBasicAuth('user', 'pass'))

# OAuth
from requests_oauthlib import OAuth2Session
oauth = OAuth2Session(client_id)
token = oauth.fetch_token(token_url, client_secret=client_secret)
```

### Beautiful Soup

[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) 是 HTML/XML 解析库。

```python
from bs4 import BeautifulSoup
import requests

# 解析 HTML
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

soup = BeautifulSoup(html_doc, 'html.parser')

# 基本导航
print(soup.title)
print(soup.title.string)
print(soup.title.parent.name)
print(soup.p)
print(soup.p['class'])
print(soup.a)
print(soup.find_all('a'))

# 查找元素
print(soup.find('p', class_='title'))
print(soup.find_all('a', class_='sister'))
print(soup.find_all(id='link2'))

# CSS 选择器
print(soup.select('p.title'))
print(soup.select('a#link1'))
print(soup.select('p.story > a'))

# 获取属性
link = soup.find('a')
print(link.get('href'))
print(link['href'])

# 获取文本
print(soup.get_text())
print(soup.find('p').get_text())

# 修改文档
tag = soup.find('p', class_='title')
tag.string = 'New title'
tag['class'] = 'new-class'

# 删除元素
tag.clear()

# 实际爬虫示例
def scrape_website(url):
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # 提取标题
    title = soup.find('h1', class_='title')
    
    # 提取所有链接
    links = []
    for link in soup.find_all('a', href=True):
        links.append({
            'text': link.get_text(strip=True),
            'url': link['href']
        })
    
    # 提取表格数据
    tables = []
    for table in soup.find_all('table'):
        rows = []
        for tr in table.find_all('tr'):
            row = [td.get_text(strip=True) for td in tr.find_all(['td', 'th'])]
            rows.append(row)
        tables.append(rows)
    
    return {
        'title': title.get_text() if title else None,
        'links': links,
        'tables': tables
    }

# 解析 XML
xml_doc = """
<root>
    <item id="1">
        <name>Item 1</name>
        <price>10.99</price>
    </item>
    <item id="2">
        <name>Item 2</name>
        <price>20.99</price>
    </item>
</root>
"""

soup = BeautifulSoup(xml_doc, 'xml')
for item in soup.find_all('item'):
    print(f"ID: {item['id']}, Name: {item.find('name').string}")
```

### Selenium

[Selenium](https://www.selenium.dev/) 是浏览器自动化工具。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

# 基本设置
options = Options()
options.add_argument('--headless')  # 无头模式
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')

# 启动浏览器
driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)

# 打开网页
driver.get('https://www.google.com')

# 查找元素
search_box = driver.find_element(By.NAME, 'q')
search_box.send_keys('Python Selenium')
search_box.send_keys(Keys.RETURN)

# 等待元素加载
wait = WebDriverWait(driver, 10)
results = wait.until(EC.presence_of_all_elements_located((By.CSS_SELECTOR, 'h3')))

# 提取结果
for result in results[:5]:
    print(result.text)

# 截图
driver.save_screenshot('screenshot.png')

# 执行 JavaScript
driver.execute_script('window.scrollTo(0, document.body.scrollHeight);')

# 处理弹窗
alert = driver.switch_to.alert
alert.accept()  # 或 alert.dismiss()

# 切换窗口
driver.switch_to.window(driver.window_handles[1])

# 切换 frame
driver.switch_to.frame('frame_name')
driver.switch_to.default_content()

# 操作示例：登录
def login(username, password):
    driver.get('https://example.com/login')
    
    driver.find_element(By.ID, 'username').send_keys(username)
    driver.find_element(By.ID, 'password').send_keys(password)
    driver.find_element(By.ID, 'login-button').click()
    
    # 等待登录完成
    wait.until(EC.presence_of_element_located((By.ID, 'dashboard')))

# 操作示例：表单填写
def fill_form():
    driver.get('https://example.com/form')
    
    # 文本输入
    driver.find_element(By.ID, 'name').send_keys('张三')
    
    # 下拉选择
    from selenium.webdriver.support.ui import Select
    select = Select(driver.find_element(By.ID, 'country'))
    select.select_by_value('CN')
    
    # 单选按钮
    driver.find_element(By.CSS_SELECTOR, 'input[value="male"]').click()
    
    # 复选框
    checkbox = driver.find_element(By.ID, 'agree')
    if not checkbox.is_selected():
        checkbox.click()
    
    # 文件上传
    driver.find_element(By.ID, 'file').send_keys('/path/to/file.txt')
    
    # 提交表单
    driver.find_element(By.ID, 'submit').click()

# 操作示例：滚动加载
def scroll_load():
    driver.get('https://example.com/infinite-scroll')
    
    last_height = driver.execute_script('return document.body.scrollHeight')
    
    while True:
        driver.execute_script('window.scrollTo(0, document.body.scrollHeight);')
        time.sleep(2)
        
        new_height = driver.execute_script('return document.body.scrollHeight')
        if new_height == last_height:
            break
        last_height = new_height

# Cookie 操作
driver.add_cookie({'name': 'key', 'value': 'value'})
cookies = driver.get_cookies()
driver.delete_cookie('key')
driver.delete_all_cookies()

# 关闭浏览器
driver.quit()
```

## 数据库

### SQLAlchemy

[SQLAlchemy](https://www.sqlalchemy.org/) 是 Python SQL 工具包和 ORM。

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, ForeignKey, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship, joinedload
from sqlalchemy.sql import func
from datetime import datetime

Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(80), unique=True, nullable=False)
    email = Column(String(120), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # 关系
    posts = relationship('Post', back_populates='author', lazy='dynamic')
    
    def __repr__(self):
        return f'<User {self.username}>'

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text)
    user_id = Column(Integer, ForeignKey('users.id'), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # 关系
    author = relationship('User', back_populates='posts')
    
    def __repr__(self):
        return f'<Post {self.title}>'

# 创建引擎
engine = create_engine('sqlite:///app.db', echo=True)
# engine = create_engine('postgresql://user:password@localhost/dbname')
# engine = create_engine('mysql+pymysql://user:password@localhost/dbname')

# 创建表
Base.metadata.create_all(engine)

# 创建会话
Session = sessionmaker(bind=engine)
session = Session()

# 创建记录
new_user = User(username='张三', email='zhangsan@example.com')
session.add(new_user)
session.commit()

# 批量插入
users = [
    User(username='李四', email='lisi@example.com'),
    User(username='王五', email='wangwu@example.com'),
]
session.bulk_save_objects(users)
session.commit()

# 查询记录
user = session.query(User).filter_by(username='张三').first()
user = session.query(User).filter(User.username == '张三').first()

# 查询所有
all_users = session.query(User).all()

# 条件查询
active_users = session.query(User).filter(User.created_at > datetime(2023, 1, 1)).all()

# 模糊查询
users = session.query(User).filter(User.username.like('%张%')).all()

# 排序
users = session.query(User).order_by(User.created_at.desc()).all()

# 分页
users = session.query(User).offset(10).limit(10).all()

# 聚合
from sqlalchemy import func
count = session.query(func.count(User.id)).scalar()
avg_id = session.query(func.avg(User.id)).scalar()

# 连接查询
posts_with_authors = session.query(Post, User).join(User).all()

# 预加载关系
posts = session.query(Post).options(joinedload(Post.author)).all()

# 更新记录
user = session.query(User).filter_by(username='张三').first()
user.email = 'newemail@example.com'
session.commit()

# 批量更新
session.query(User).filter(User.username == '张三').update({
    'email': 'updated@example.com'
})
session.commit()

# 删除记录
session.delete(user)
session.commit()

# 事务处理
try:
    user1 = User(username='user1', email='user1@example.com')
    user2 = User(username='user2', email='user2@example.com')
    session.add(user1)
    session.add(user2)
    session.commit()
except:
    session.rollback()
    raise
finally:
    session.close()

# 使用上下文管理器
with Session() as session:
    user = session.query(User).first()
    print(user)
    # 自动提交和关闭

# 原生 SQL
result = session.execute("SELECT * FROM users WHERE username = :name", {'name': '张三'})
for row in result:
    print(row)
```

### Redis-py

[Redis-py](https://github.com/redis/redis-py) 是 Redis 的 Python 客户端。

```python
import redis
from redis.exceptions import RedisError
import json

# 连接 Redis
r = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    password=None,
    decode_responses=True  # 自动解码为字符串
)

# 连接池
pool = redis.ConnectionPool(host='localhost', port=6379, db=0)
r = redis.Redis(connection_pool=pool)

# 基本操作
r.set('key', 'value')
value = r.get('key')
print(value)

# 设置过期时间
r.setex('temp_key', 3600, 'temp_value')  # 1小时过期
r.expire('key', 300)  # 5分钟后过期

# 批量操作
r.mset({'key1': 'value1', 'key2': 'value2'})
values = r.mget(['key1', 'key2'])

# 递增/递减
r.set('counter', 0)
r.incr('counter')      # +1
r.incrby('counter', 5) # +5
r.decr('counter')      # -1

# 哈希操作
r.hset('user:1', 'name', '张三')
r.hset('user:1', 'age', 25)
r.hmset('user:1', {'email': 'zhangsan@example.com', 'city': '北京'})

user = r.hgetall('user:1')
name = r.hget('user:1', 'name')
age = r.hincrby('user:1', 'age', 1)  # 年龄+1

# 列表操作
r.lpush('mylist', 'item1')
.rpush('mylist', 'item2')
items = r.lrange('mylist', 0, -1)
item = r.lpop('mylist')
r.lrem('mylist', 1, 'item1')  # 删除1个'item1'

# 集合操作
r.sadd('myset', 'member1', 'member2', 'member3')
members = r.smembers('myset')
is_member = r.sismember('myset', 'member1')
r.srem('myset', 'member1')

# 有序集合
r.zadd('leaderboard', {'player1': 100, 'player2': 200, 'player3': 150})
top_players = r.zrevrange('leaderboard', 0, 2, withscores=True)
r.zincrby('leaderboard', 50, 'player1')  # 增加分数

# 发布订阅
pubsub = r.pubsub()
pubsub.subscribe('my-channel')

# 发布消息
r.publish('my-channel', 'Hello, subscribers!')

# 接收消息
for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"Received: {message['data']}")

# 事务
pipe = r.pipeline()
pipe.multi()
pipe.incr('counter')
pipe.incr('counter')
pipe.execute()

# 存储 Python 对象
data = {'name': '张三', 'age': 25, 'hobbies': ['reading', 'coding']}
r.set('user_data', json.dumps(data))
loaded_data = json.loads(r.get('user_data'))

# 分布式锁
import time
import uuid

def acquire_lock(lock_name, acquire_time=10):
    identifier = str(uuid.uuid4())
    end_time = time.time() + acquire_time
    
    while time.time() < end_time:
        if r.setnx(lock_name, identifier):
            r.expire(lock_name, acquire_time)
            return identifier
        time.sleep(0.001)
    
    return False

def release_lock(lock_name, identifier):
    pipe = r.pipeline(True)
    while True:
        try:
            pipe.watch(lock_name)
            if pipe.get(lock_name) == identifier:
                pipe.multi()
                pipe.delete(lock_name)
                pipe.execute()
                return True
            pipe.unwatch()
            break
        except redis.WatchError:
            continue
    return False

# 使用锁
lock_id = acquire_lock('my_lock')
if lock_id:
    try:
        # 执行需要同步的操作
        pass
    finally:
        release_lock('my_lock', lock_id)
```

## 工具类

### Click

[Click](https://click.palletsprojects.com/) 是创建命令行接口的 Python 包。

```python
import click
from typing import Optional

@click.group()
def cli():
    """My CLI Tool"""
    pass

@cli.command()
@click.argument('name')
@click.option('--count', '-c', default=1, help='Number of greetings.')
@click.option('--uppercase', '-u', is_flag=True, help='Print in uppercase.')
def hello(name: str, count: int, uppercase: bool):
    """Say hello to someone."""
    for _ in range(count):
        message = f"Hello, {name}!"
        if uppercase:
            message = message.upper()
        click.echo(message)

@cli.command()
@click.argument('input_file', type=click.File('r'))
@click.argument('output_file', type=click.File('w'))
@click.option('--lines', '-n', type=int, help='Number of lines to copy.')
def copy(input_file, output_file, lines: Optional[int]):
    """Copy file content."""
    if lines:
        for i, line in enumerate(input_file):
            if i >= lines:
                break
            output_file.write(line)
    else:
        output_file.write(input_file.read())
    click.echo(f"Copied to {output_file.name}")

@cli.command()
@click.confirmation_option(prompt='Are you sure you want to delete?')
@click.argument('filename')
def delete(filename: str):
    """Delete a file with confirmation."""
    import os
    os.remove(filename)
    click.echo(f"Deleted {filename}")

@cli.command()
@click.password_option(help='Enter your password.')
def login(password: str):
    """Login with password."""
    click.echo(f"Password entered: {'*' * len(password)}")

@cli.command()
@click.option('--choice', type=click.Choice(['option1', 'option2', 'option3']))
def choose(choice: str):
    """Make a choice."""
    click.echo(f"You chose: {choice}")

@cli.command()
@click.option('--config', '-c', type=click.Path(exists=True))
def config(config: Optional[str]):
    """Load configuration."""
    if config:
        click.echo(f"Loading config from {config}")
    else:
        click.echo("No config file specified")

# 进度条
@cli.command()
@click.option('--count', default=100)
def process(count: int):
    """Process items with progress bar."""
    import time
    
    with click.progressbar(range(count), label='Processing') as bar:
        for item in bar:
            time.sleep(0.01)

# 颜色输出
@cli.command()
def colored():
    """Print colored text."""
    click.echo(click.style('Hello World!', fg='green'))
    click.echo(click.style('Error!', fg='red', bold=True))
    click.echo(click.style('Warning!', fg='yellow'))

if __name__ == '__main__':
    cli()
```

### Rich

[Rich](https://github.com/Textualize/rich) 是用于在终端中渲染富文本的 Python 库。

```python
from rich.console import Console
from rich.table import Table
from rich.panel import Panel
from rich.progress import Progress, SpinnerColumn, TextColumn
from rich.syntax import Syntax
from rich.tree import Tree
from rich import print
import time

console = Console()

# 基本输出
console.print("Hello", style="bold red")
console.print("World", style="bold green")

# 表格
table = Table(title="User Information")
table.add_column("Name", style="cyan", no_wrap=True)
table.add_column("Age", style="magenta")
table.add_column("City", style="green")

table.add_row("张三", "25", "北京")
table.add_row("李四", "30", "上海")
table.add_row("王五", "28", "广州")

console.print(table)

# 面板
panel = Panel.fit(
    "[bold green]Success![/bold green] Operation completed.",
    title="Status",
    border_style="green"
)
console.print(panel)

# 语法高亮
code = '''
def hello(name):
    print(f"Hello, {name}!")

hello("World")
'''
syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)

# 进度条
with Progress(
    SpinnerColumn(),
    TextColumn("[progress.description]{task.description}"),
    transient=True,
) as progress:
    task = progress.add_task("Processing...", total=None)
    time.sleep(2)
    progress.update(task, completed=True)

# 带进度条的循环
with Progress() as progress:
    task = progress.add_task("[green]Downloading...", total=100)
    
    while not progress.finished:
        progress.update(task, advance=0.5)
        time.sleep(0.01)

# 树形结构
tree = Tree("[bold cyan]Project")
tree.add("[green]src")
tree.add("[blue]docs")
tests = tree.add("[yellow]tests")
tests.add("test_main.py")
tests.add("test_utils.py")

console.print(tree)

# 日志样式
console.log("Starting process...")
console.log("Processing item 1")
console.log("Processing item 2")
console.log("[bold green]Process completed![/bold green]")

# 富文本打印
print("[bold red]Alert![/bold red] Something went [italic]wrong[/italic]")

# 状态显示
with console.status("[bold green]Working...") as status:
    time.sleep(2)
    console.print("[bold green]Done!")

# 多列布局
from rich.columns import Columns

users = ["User 1", "User 2", "User 3", "User 4", "User 5"]
columns = Columns(users, equal=True, expand=True)
console.print(columns)
```

### Pydantic

[Pydantic](https://docs.pydantic.dev/) 是使用 Python 类型提示进行数据验证的库。

```python
from pydantic import BaseModel, Field, validator, EmailStr, HttpUrl
from typing import List, Optional, Literal
from datetime import datetime
from enum import Enum

class Gender(str, Enum):
    MALE = "male"
    FEMALE = "female"
    OTHER = "other"

class Address(BaseModel):
    street: str
    city: str
    country: str = "China"
    postal_code: str = Field(..., regex=r'^\d{6}$')

class User(BaseModel):
    id: int = Field(..., gt=0, description="User ID")
    name: str = Field(..., min_length=2, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
    gender: Gender = Gender.OTHER
    is_active: bool = True
    address: Optional[Address] = None
    tags: List[str] = []
    website: Optional[HttpUrl] = None
    created_at: datetime = Field(default_factory=datetime.now)
    
    @validator('name')
    def name_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Name must not be empty')
        return v.title()
    
    @validator('tags', each_item=True)
    def tags_must_be_lowercase(cls, v):
        return v.lower()
    
    class Config:
        json_schema_extra = {
            "example": {
                "id": 1,
                "name": "张三",
                "email": "zhangsan@example.com",
                "age": 25,
                "gender": "male",
                "is_active": True,
                "tags": ["developer", "python"]
            }
        }

# 创建实例
try:
    user = User(
        id=1,
        name="zhang san",
        email="zhangsan@example.com",
        age=25,
        gender=Gender.MALE,
        address=Address(
            street="中关村大街1号",
            city="北京",
            postal_code="100080"
        ),
        tags=["Developer", "Python"],
        website="https://example.com"
    )
    print(user)
except Exception as e:
    print(f"Validation error: {e}")

# 从字典创建
data = {
    "id": 2,
    "name": "李四",
    "email": "lisi@example.com",
    "age": 30,
    "tags": ["designer"]
}
user2 = User(**data)

# 从 JSON 创建
import json
json_data = '{"id": 3, "name": "王五", "email": "wangwu@example.com", "age": 28}'
user3 = User.parse_raw(json_data)

# 导出为 JSON
print(user.json(indent=2))
print(user.dict())

# 部分更新
user_data = user.dict()
user_data['age'] = 26
updated_user = User(**user_data)

# 嵌套模型
class Team(BaseModel):
    name: str
    members: List[User]
    
team = Team(
    name="开发团队",
    members=[user, user2]
)

# 自定义验证器
from pydantic import root_validator

class Order(BaseModel):
    price: float = Field(..., gt=0)
    quantity: int = Field(..., gt=0)
    total: float = 0
    
    @root_validator
    def calculate_total(cls, values):
        values['total'] = values['price'] * values['quantity']
        return values

order = Order(price=10.5, quantity=3)
print(order.total)  # 31.5

# 设置管理
from pydantic import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My App"
    debug: bool = False
    database_url: str
    secret_key: str
    
    class Config:
        env_file = '.env'

settings = Settings()
print(settings.database_url)
```

## 总结

本文介绍了 Python 生态中常用的第三方库，涵盖了：

- **Web 开发**：Django、Flask、FastAPI
- **数据科学**：NumPy、Pandas、Matplotlib、Seaborn
- **机器学习**：Scikit-learn、TensorFlow/Keras、PyTorch
- **自动化**：Requests、Beautiful Soup、Selenium
- **数据库**：SQLAlchemy、Redis-py
- **工具类**：Click、Rich、Pydantic

这些库都经过广泛使用和验证，可以帮助你更高效地开发 Python 应用。建议根据项目需求选择合适的库，并始终关注官方文档获取最新信息。

想象无线，都来 Python! 🐍

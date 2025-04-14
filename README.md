Project Structure:
fastapi-app/
│
├── main.py
├── models.py
├── database.py
└── requirements.txt
1. Install the Necessary Dependencies:
In your requirements.txt, list the following dependencies:
fastapi
uvicorn
psycopg2-binary
sqlalchemy
pydantic
You can install them by running:
pip install -r requirements.txt
2. Database Setup (database.py):
Create a file named database.py to handle the connection to your PostgreSQL database.
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
# Using environment variable for database URL
database_url = os.environ.get("DATABASE_URL")
# Creating the engine to interact with PostgreSQL
engine = create_engine(database_url, echo=True)
# Creating a session local class to interact with the database
SessionLocal = sessionmaker(autocommit=False, autoflush=False,
bind=engine)
# Base class to create tables
Base = declarative_base()
3. Model Definition (models.py):
In models.py, define the Book table using SQLAlchemy.
from sqlalchemy import Column, Integer, String
from database import Base
class Book(Base):
__tablename__ = "books"
id = Column(Integer, primary_key=True, index=True)
title = Column(String, index=True)
author = Column(String)
This defines a Book table with three fields: id, title, and author.
4. FastAPI App (main.py):
Update main.py to integrate PostgreSQL and SQLAlchemy for your FastAPI application.
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from pydantic import BaseModel
from typing import List
from database import SessionLocal, engine
from models import Book, Base
# Create tables in the database
Base.metadata.create_all(bind=engine)
app = FastAPI()
# Dependency to get the database session
def get_db():
db = SessionLocal()
try:
yield db
finally:
db.close()
# Pydantic model to use in FastAPI request and response
class BookCreate(BaseModel):
title: str
author: str
class BookOut(BookCreate):
id: int
class Config:
orm_mode = True
# Endpoint to fetch all books
@app.get("/books", response_model=List[BookOut])
def get_books(db: Session = Depends(get_db)):
books = db.query(Book).all()
return books
# Endpoint to add a new book
@app.post("/books", response_model=BookOut)
def add_book(book: BookCreate, db: Session = Depends(get_db)):
db_book = Book(title=book.title, author=book.author)
db.add(db_book)
db.commit()
db.refresh(db_book)
return db_book
Key Elements:
• SessionLocal: Used to interact with the PostgreSQL database.
• get_db(): Dependency that provides the database session to the endpoints.
• BookCreate and BookOut: Pydantic models to validate and serialize the request/response.
5. Deploy to Render:
• Create a New Web Service:
o In the Render dashboard, click on Create New → Web Service.
o Connect your GitHub repository containing your FastAPI app.
o Select the branch to deploy (usually main or master).
o Set the Environment to Python (Render auto-detects this).
o In the Build Command field, use:
pip install -r requirements.txt
o In the Start Command field, use:
uvicorn main:app --host 0.0.0.0 --port 8000
• Set the PostgreSQL Database URL in Render:
o In Render, after creating the PostgreSQL database, you'll get a DATABASE_URL that
looks like this:
postgresql://username:password@host:port/database_name
o In your Render dashboard, go to your Web Service settings, add the DATABASE_URL as
an Environment Variable:
Name: DATABASE_URL
Value: Paste the INTERNAL DATABASE URL provided by Render.
• Deploy:
o Click Create Web Service to deploy your app. Render will automatically pull your
code, install dependencies, and start your FastAPI app.
6. Access the Deployed App
Once deployed, Render will provide a URL where your FastAPI app is live. something like:
https://your-app-name.onrender.com
You can now visit this URL to test the app and start with the FastAPI documentation at:
https://your-app-name.onrender.com/docs

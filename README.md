Project Structure:
fastapi-app/
│
├── main.py
├── models.py
├── database.py
└── requirements.txt

Install the Necessary Dependencies:
In your requirements.txt, list the following dependencies:

fastapi
uvicorn
psycopg2-binary
sqlalchemy
pydantic

Install them with:
pip install -r requirements.txt

Database Setup (database.py):
Create a file named database.py to handle the connection to your PostgreSQL database:

import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

# Using environment variable for database URL
database_url = os.environ.get("DATABASE_URL")

# Creating the engine to interact with PostgreSQL
engine = create_engine(database_url, echo=True)

# Creating a session local class to interact with the database
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base class to create tables
Base = declarative_base()
3. Model Definition (models.py):
Define the Book table using SQLAlchemy:

from sqlalchemy import Column, Integer, String
from database import Base

class Book(Base):
    __tablename__ = "books"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    author = Column(String)
FastAPI App (main.py):
Integrate PostgreSQL and SQLAlchemy in your FastAPI application:

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
SessionLocal: Used to interact with the PostgreSQL database.

get_db(): Dependency that provides the database session to the endpoints.

BookCreate and BookOut: Pydantic models to validate and serialize the request/response.

Deploy to Render:
• Create a New Web Service:
Go to the Render dashboard.

Click Create New → Web Service.

Connect your GitHub repository containing your FastAPI app.

Select the branch to deploy (usually main or master).

Set the environment to Python (Render auto-detects this).

Build Command:

pip install -r requirements.txt
Start Command:

uvicorn main:app --host 0.0.0.0 --port 8000
• Set the PostgreSQL Database URL in Render:
After creating the PostgreSQL database, Render provides a DATABASE_URL, which looks like:

postgresql://username:password@host:port/database_name
In your Render dashboard, go to your Web Service settings and add the following Environment Variable:

Name: DATABASE_URL
Value: (Paste the INTERNAL DATABASE URL provided by Render)


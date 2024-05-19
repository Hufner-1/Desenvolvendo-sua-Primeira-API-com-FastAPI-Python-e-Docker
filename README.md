Fast API 

       from fastapi import FastAPI, HTTPException
       from sqlalchemy.exc import IntegrityError
       from typing import Optional
       from fastapi_pagination import Page, paginate
       from fastapi_pagination.ext.sqlalchemy import paginate
       from pydantic import BaseModel
       from sqlalchemy import create_engine, Column, Integer, String
       from sqlalchemy.orm import sessionmaker
       from sqlalchemy.ext.declarative import declarative_base

       app = FastAPI()

       # Configuração do banco de dados
       DATABASE_URL = "sqlite:///./test.db"
       engine = create_engine(DATABASE_URL)
       SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

       Base = declarative_base()

       class Atleta(Base):
    __tablename__ = "atletas"

    id = Column(Integer, primary_key=True, index=True)
    nome = Column(String)
    cpf = Column(String, unique=True)
    centro_treinamento = Column(String)
    categoria = Column(String)

       Base.metadata.create_all(bind=engine)

       class AtletaCreate(BaseModel):
    nome: str
    cpf: str
    centro_treinamento: str
    categoria: str

       @app.get("/atleta", response_model=Page[Atleta])
       async def read_atleta(nome: Optional[str] = None, cpf: Optional[str] = None):
    db = SessionLocal()
    atletas = db.query(Atleta)
    if nome:
        atletas = atletas.filter(Atleta.nome == nome)
    if cpf:
        atletas = atletas.filter(Atleta.cpf == cpf)
    return paginate(atletas)

       @app.post("/atleta")
       async def create_atleta(atleta: AtletaCreate):
    db = SessionLocal()
    try:
        db_atleta = Atleta(**atleta.dict())
        db.add(db_atleta)
        db.commit()
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=303, detail="Já existe um atleta cadastrado com o cpf: x")
    finally:
        db.close()

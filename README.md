import sqlite3
from typing import List, Tuple, Optional

class DatabaseManager:
    def __init__(self, db_name: str = "controle_estoque.db"):
        self.db_name = db_name
        self.init_db()

    def get_connection(self):
        return sqlite3.connect(self.db_name)

    def init_db(self):
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                CREATE TABLE IF NOT EXISTS produtos (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    codigo TEXT UNIQUE NOT NULL,
                    nome TEXT NOT NULL,
                    quantidade INTEGER NOT NULL CHECK(quantidade >= 0),
                    estoque_minimo INTEGER NOT NULL DEFAULT 5,
                    preco_unitario REAL NOT NULL
                )
            """)
            cursor.execute("""
                CREATE TABLE IF NOT EXISTS movimentacoes (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    produto_id INTEGER NOT NULL,
                    tipo TEXT CHECK(tipo IN ('ENTRADA', 'SAIDA')) NOT NULL,
                    quantidade INTEGER NOT NULL,
                    data_hora DATETIME DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (produto_id) REFERENCES produtos (id)
                )
            """)
            conn.commit()

    def inserir_produto(self, codigo: str, nome: str, qtd: int, min_qtd: int, preco: float) -> bool:
        try:
            with self.get_connection() as conn:
                cursor = conn.cursor()
                cursor.execute(
                    "INSERT INTO produtos (codigo, nome, quantidade, estoque_minimo, preco_unitario) VALUES (?, ?, ?, ?, ?)",
                    (codigo, nome, qtd, min_qtd, preco)
                )
                conn.commit()
                return True
        except sqlite3.IntegrityError:
            return False

    def registrar_movimentacao(self, produto_id: int, tipo: str, quantidade: int) -> bool:
        with self.get_connection() as conn:
            cursor = conn.cursor()
            
            # Checa estoque atual antes de dar saída
            if tipo == 'SAIDA':
                cursor.execute("SELECT quantidade FROM produtos WHERE id = ?", (produto_id,))
                res = cursor.fetchone()
                if not res or res[0] < quantidade:
                    return False

            fator = 1 if tipo == 'ENTRADA' else -1
            cursor.execute(
                "UPDATE produtos SET quantidade = quantidade + ? WHERE id = ?",
                (quantidade * fator, produto_id)
            )
            cursor.execute(
                "INSERT INTO movimentacoes (produto_id, tipo, quantidade) VALUES (?, ?, ?)",
                (produto_id, tipo, quantidade)
            )
            conn.commit()
            return True

    def buscar_todos_produtos(self) -> List[Tuple]:
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute("SELECT id, codigo, nome, quantidade, estoque_minimo, preco_unitario FROM produtos")
            return cursor.fetchall()[app.py](https://github.com/user-attachments/files/31892305/app.py)


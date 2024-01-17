from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
# Banco de dados
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///ecomerce.db'
# Iniciar conexao
db = SQLAlchemy(app)


#Modelagem do produto
#linha = registro
#coluna = informações produto( os campos que vou armazenar a informação)(id, name, price, description)
class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    price = db.Column(db.Float, nullable=False)
    description = db.Column(db.Text, nullable=True)

# Definir uma rota raiz (pagina inicial) e a função que sera executada ao requisitar
@app.route('/')
def hello_world():
    return 'OLA GRACI'

if __name__ == "__main__":
    app.run(debug=True)

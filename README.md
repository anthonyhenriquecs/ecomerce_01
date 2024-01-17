# ecomerce_01

from flask import Flask

app = Flask(__name__)


# Definir uma rota raiz (pagina inicial) e a função que sera executada ao requisitar
@app.route('/')
def hello_hord():
    return 'Ola mundo :0'

if __name__ == " __main__":
    app.run(debug=True)

from flask import Flask, request, jsonify
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
@app.route('/api/products/add', methods=["POST"])
def add_product():
    data = request.json
    if 'name' in data and 'price' in data:
        product = Product(name=data["name"], price=float(data["price"]), description=data.get("description", ""))
        db.session.add(product)
        db.session.commit()
        return "Produto cadastrado com sucesso"
    return jsonify({"message": "Invalid product data"}), 400

@app.route('/api/products/delete/<int:product_id>', methods=["DELETE"])
def delete_product(product_id):
    #recuperar o produto da base de dados
    product = Product.query.get(product_id)
    #Verificar se o produto existe
    if product:
        db.session.delete(product)
        db.session.commit()
        return "Produto deletado com sucesso"
    return jsonify({"message": "Product not foun"}), 404
    #Se existe, apagar da base de dados
    #Se não existe, retornar 404 not found
    
    
@app.route('/api/products/<int:product_id>', methods=["GET"])
def get_product_details(product_id):
    product = Product.query.get(product_id)
    if product:
        return jsonify({
            "id": product.id,
            "name": product.name,
            "price": product.price,
            "description": product.description
        })
    return jsonify({"message": "Product not foun"}), 404
@app.route('/api/products/update/<int:product_id>', methods=["PUT"])
def update_product(product_id):
    product = Product.query.get(product_id)
    if not product:
        return jsonify({"message": "Product not foun"}), 404
    
    data = request.json
    if 'name' in data:
        product.name = data['name']
        
    if 'price' in data:
        product.price = data['price']
        
    if 'description' in data:
        product.description = data['description']
    db.session.commit()
        
    return jsonify({'message': 'Produto atualizado com sucesso'})

@app.route('/api/products', methods=['GET'])
def get_products():
    products = Product.query.all()
    product_list = []
    for product in products:
        product_data = {
            "id": product.id,
            "name": product.name,
            "price": product.price,
            "description": product.description
        }
        product_list.append(product_data)
        
    return jsonify(product_list)

# Definir uma rota raiz (pagina inicial) e a função que sera executada ao requisitar
@app.route('/')
def hello_world():
    return 'OLA Mundo'

if __name__ == "__main__":
    app.run(debug=True)




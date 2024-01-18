from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS
from flask_login import UserMixin, login_user,LoginManager, login_required,logout_user, current_user

app = Flask(__name__)
# Banco de dados
app.config['SECRET_KEY'] = "minha_chave_123"
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///ecomerce.db'
# Iniciar conexao
login_maneger = LoginManager()
db = SQLAlchemy(app)
login_maneger.init_app(app)
login_maneger.login_view = 'login'
CORS(app)

#Modelagem 
#Usuario (id, username, passoword)
class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), nullable=False, unique=True)
    password = db.Column(db.String(80), nullable=True)
    cart = db.relationship('CartItem', backref='user', lazy=True)
    
#linha = registro
#coluna = informações produto( os campos que vou armazenar a informação)(id, name, price, description)
class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    price = db.Column(db.Float, nullable=False)
    description = db.Column(db.Text, nullable=True)
    
class CartItem(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    
    
# Autenticação
@login_maneger.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))    
@app.route('/login', methods=["POST"])
def login():
    data = request.json
    
    user = User.query.filter_by(username=data.get("username")).first()  # Corrigido para atribuir o valor a uma variável
    if user and data.get("password") == user.password:
            login_user(user)
            return jsonify({"message": "Logado com sucesso"})
        
    return jsonify({"message": "Invalid login data"}), 401

    
    
@app.route('/logout', methods=["POST"])
@login_required
def logout():
    logout_user()
    return jsonify({"message": "Deslogado com sucesso"})    
@app.route('/api/products/add', methods=["POST"])
@login_required
def add_product():
    data = request.json
    if 'name' in data and 'price' in data:
        product = Product(name=data["name"], price=float(data["price"]), description=data.get("description", ""))
        db.session.add(product)
        db.session.commit()
        return "Produto cadastrado com sucesso"
    return jsonify({"message": "Invalid product data"}), 400

@app.route('/api/products/delete/<int:product_id>', methods=["DELETE"])
@login_required
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
@login_required
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
            "price": product.price
        }
        product_list.append(product_data)
        
    return jsonify(product_list)

#Cheskout
@app.route('/api/cart/add/<int:product_id>', methods=['POST'])
@login_required
def add_to_cart(product_id):
    #Usuario
    user = User.query.get(int(current_user.id))
    #Produto
    product = Product.query.get(product_id)
    if user and product:
        cart_item = CartItem(user_id=user.id, product_id=product.id)
        db.session.add(cart_item)
        db.session.commit()
        return jsonify({'message': 'Item adicionado com sucesso ao carrinho'})
    return  jsonify({'message': 'Falha ao adicionar o item ao carrinho'}),400

@app.route('/api/cart/remove/<int:product_id>', methods=['DELETE'])
@login_required
def remove_from_cart(product_id):
    #Produto, Usuario = Item no carrinho
    cart_item = CartItem.query.filter_by(user_id=current_user.id, product_id=product_id).first()
    if cart_item:
        db.session.delete(cart_item)
        db.session.commit()
        return jsonify({'message': 'Item removido com sucesso do carrinho'})
    return jsonify({'message':'Falha ao remover item do carrinho'}),400

@app.route('/api/cart', methods=['GET'])
@login_required
def view_cart():
    #usuario
    user = User.query.get(int(current_user.id))
    cart_items = user.cart
    cart_content = []
    for cart_item in cart_items:
        product = Product.query.get(cart_item.product_id)
        cart_content.append({
                                "id": cart_item.id,
                                "user_id": cart_item.user_id,
                                "product_id": cart_item.product_id,
                                "product_name": product.name,
                                "product_price": product.price
                            })
    return jsonify(cart_content)

@app.route('/api/cart/checkout', methods=["POST"])
@login_required
def checkout():
    user = User.query.get(int(current_user.id))
    cart_items = user.cart
    for cart_item in cart_items:
        db.session.delete(cart_item)
    db.session.commit()
    return jsonify({'message': 'Check-out bem-sucedido. O carrinho foi limpo'})

if __name__ == "__main__":
    app.run(debug=True)




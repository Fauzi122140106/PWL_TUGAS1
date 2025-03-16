**Langkah pertama lakukan clone untuk reponya :**
git clone https://github.com/woocommerce/woocommerce.git
cd woocommerce

**Langkah kedua install WordPress dan WooCommerce:**
Jalankan layanan WordPress dan MySQL dengan Docker:
version: "3.8"
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DB_NAME: woocommerce
    volumes:
      - ./woocommerce:/var/www/html/wp-content/plugins/woocommerce

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: woocommerce
    ports:
      - "3306:3306"

  Jalankan dengan: 
    docker-compose up -d

**Langkah ketiga Migrasi ke Microservices** : 
Membuat Product Service (Node.js + MongoDB)

mkdir product-service && cd product-service
npm init -y
npm install express mongoose dotenv cors

**Langkah keempat buatlah server Node.JSnya**
const express = require("express");
const mongoose = require("mongoose");
require("dotenv").config();

const app = express();
app.use(express.json());

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });

const ProductSchema = new mongoose.Schema({
  name: String,
  price: Number,
  stock: Number,
});
const Product = mongoose.model("Product", ProductSchema);

app.get("/products", async (req, res) => {
  const products = await Product.find();
  res.json(products);
});

app.listen(5001, () => console.log("Product Service running on port 5001"));

**Jalankan dengan :**
node server.js

Langkah kelima **Membuat Order Service (Spring Boot + PostgreSQL)**
**Buat orde java **
@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    private String userId;
    private Double totalAmount;
}
**Buat OrderController.java**
@RestController
@RequestMapping("/orders")
public class OrderController {
    @Autowired private OrderService service;

  @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        return ResponseEntity.ok(service.createOrder(order));
    }
}

**Jalankan Order Service**
mvn spring-boot:run

**Langkah keenam Setup API Gateway dengan Nginx**
Buat file nginx.conf:
server {
    listen 80;

    location /products/ {
        proxy_pass http://product-service:5001/;
    }

    location /orders/ {
        proxy_pass http://order-service:8080/;
    }
}

Jalankan dengan Docker:
docker run -d --name api-gateway -p 80:80 -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx

 Langkah ketujuh Deployment dengan Docker Compose :
version: "3.8"
services:
  product-service:
    build: ./product-service
    ports:
      - "5001:5001"
    depends_on:
      - mongo
  order-service:
    build: ./order-service
    ports:
      - "8080:8080"
    depends_on:
      - postgres
  api-gateway:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
  mongo:
    image: mongo
    ports:
      - "27017:27017"
  postgres:
    image: postgres
    environment:
      POSTGRES_DB: orderdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: yourpassword
 
jalankan dengan :
 docker-compose up -d


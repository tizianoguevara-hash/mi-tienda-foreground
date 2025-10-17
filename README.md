<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Foreground - Tienda Online</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; }
        header { background-color: #333; color: white; padding: 10px; text-align: center; }
        .products { display: flex; flex-wrap: wrap; justify-content: center; padding: 20px; }
        .product { border: 1px solid #ddd; margin: 10px; padding: 10px; width: 200px; text-align: center; }
        .cart { position: fixed; top: 10px; right: 10px; background: #f9f9f9; border: 1px solid #ddd; padding: 10px; width: 300px; }
        .admin-panel { display: none; background: #eee; padding: 20px; margin: 20px; }
        .whatsapp-icon { position: fixed; bottom: 20px; right: 20px; background: green; color: white; padding: 10px; border-radius: 50%; cursor: pointer; }
        .maintenance { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); color: white; text-align: center; padding-top: 20%; font-size: 24px; }
    </style>
</head>
<body>
    <header>
        <h1>Foregeound - Tienda de Ropa</h1>
    </header>

    <div id="maintenance" class="maintenance">Sitio en mantenimiento. Vuelve pronto.</div>

    <div id="main-content">
        <section class="products">
            <!-- Productos se cargan dinámicamente desde JavaScript -->
        </section>

        <div class="cart">
            <h2>Carrito</h2>
            <div id="cart-items"></div>
            <p>Total: <span id="cart-total">0 ARS</span></p>
            <label><input type="checkbox" id="shipping"> Agregar envío a Rosario, Santa Fe (5000 ARS)</label>
            <p>Métodos de pago:
                <select id="payment-method">
                    <option value="transferencia">Transferencia bancaria</option>
                    <option value="efectivo">Efectivo</option>
                </select>
            </p>
            <button onclick="checkout()">Finalizar compra</button>
        </div>

        <button onclick="showAdminLogin()">Acceso Administrador</button>

        <div id="admin-login" style="display: none;">
            <h2>Ingresar como Administrador</h2>
            <input type="password" id="admin-password" placeholder="Contraseña">
            <button onclick="loginAdmin()">Ingresar</button>
        </div>

        <div id="admin-panel" class="admin-panel">
            <h2>Panel de Administrador</h2>
            <button onclick="logoutAdmin()">Cerrar Sesión</button>
            <h3>Editar Productos</h3>
            <div id="admin-products"></div>
            <button onclick="saveProducts()">Guardar Cambios</button>
            <h3>Códigos de Descuento</h3>
            <input type="text" id="discount-code-name" placeholder="Nombre del código">
            <input type="number" id="discount-percentage" placeholder="Porcentaje de descuento (%)">
            <button onclick="addDiscount()">Agregar Código</button>
            <ul id="discount-list"></ul>
            <h3>Cerrar Página</h3>
            <button onclick="closeSite()">Cerrar Sitio (Modo Mantenimiento)</button>
            <button onclick="openSite()">Abrir Sitio</button>
        </div>
    </div>

    <div class="whatsapp-icon" onclick="openWhatsApp()">💬</div>

    <script>
        // Datos iniciales (se guardan en localStorage)
        let products = JSON.parse(localStorage.getItem('products')) || [
            { id: 1, name: 'Remera Básica', price: 2000, image: 'https://via.placeholder.com/150?text=Remera' },
            { id: 2, name: 'Pantalón Jeans', price: 5000, image: 'https://via.placeholder.com/150?text=Pantalon' },
            { id: 3, name: 'Buzo Deportivo', price: 3000, image: 'https://via.placeholder.com/150?text=Buzo' }
        ];

        let discounts = JSON.parse(localStorage.getItem('discounts')) || []; // Lista de códigos
        let cart = []; // Carrito temporal
        let siteClosed = localStorage.getItem('siteClosed') === 'true'; // Estado del sitio

        if (siteClosed) {
            document.getElementById('maintenance').style.display = 'block';
            document.getElementById('main-content').style.display = 'none';
        }

        function renderProducts() {
            const productsSection = document.querySelector('.products');
            productsSection.innerHTML = '';
            products.forEach(product => {
                const productDiv = document.createElement('div');
                productDiv.className = 'product';
                productDiv.innerHTML = `
                    <img src="${product.image}" alt="${product.name}" width="150">
                    <h3>${product.name}</h3>
                    <p>Precio: ${product.price} ARS</p>
                    <button onclick="addToCart(${product.id})">Agregar al Carrito</button>
                `;
                productsSection.appendChild(productDiv);
            });
        }

        function renderAdminProducts() {
            const adminProductsDiv = document.getElementById('admin-products');
            adminProductsDiv.innerHTML = '';
            products.forEach((product, index) => {
                const productEdit = document.createElement('div');
                productEdit.innerHTML = `
                    <h4>${product.name}</h4>
                    <input type="number" value="${product.price}" id="admin-price-${index}">
                    <input type="text" value="${product.image}" id="admin-image-${index}" placeholder="URL de imagen">
                    <button onclick="updateProduct(${index})">Actualizar</button>
                    <button onclick="deleteProduct(${index})">Eliminar</button>
                `;
                adminProductsDiv.appendChild(productEdit);
            });
            // Agregar opción para nuevo producto
            const newProductInput = document.createElement('div');
            newProductInput.innerHTML = `
                <h4>Agregar Nuevo Producto</h4>
                <input type="text" id="new-name" placeholder="Nombre">
                <input type="number" id="new-price" placeholder="Precio">
                <input type="text" id="new-image" placeholder="URL de imagen">
                <button onclick="addNewProduct()">Agregar</button>
            `;
            adminProductsDiv.appendChild(newProductInput);
        }

        function addToCart(productId) {
            const product = products.find(p => p.id === productId);
            cart.push(product);
            updateCart();
        }

        function updateCart() {
            const cartItemsDiv = document.getElementById('cart-items');
            cartItemsDiv.innerHTML = '';
            let total = 0;
            cart.forEach(item => {
                cartItemsDiv.innerHTML += `<p>${item.name} - ${item.price} ARS</p>`;
                total += item.price;
            });
            if (document.getElementById('shipping').checked) {
                total += 5000; // Costo de envío
            }
            document.getElementById('cart-total').textContent = total + ' ARS';
        }

        function checkout() {
            const method = document.getElementById('payment-method').value;
            const shipping = document.getElementById('shipping').checked ? 'con envío' : 'sin envío';
            alert(`Compra finalizada. Método: ${method}. Envío: ${shipping}. Total: ${document.getElementById('cart-total').textContent}`);
            // En un sitio real, redirigirías a un pago o formulario
        }

        function showAdminLogin() {
            document.getElementById('admin-login').style.display = 'block';
        }

        function loginAdmin() {
            const password = document.getElementById('admin-password').value;
            if (password === 'foreground 1924') {
                document.getElementById('admin-login').style.display = 'none';
                document.getElementById('admin-panel').style.display = 'block';
                renderAdminProducts();
                renderDiscounts();
            } else {
                alert('Contraseña incorrecta');
            }
        }

        function logoutAdmin() {
            document.getElementById('admin-panel').style.display = 'none';
        }

        function updateProduct(index) {
            products[index].price = parseFloat(document.getElementById(`admin-price-${index}`).value);
            products[index].image = document.getElementById(`admin-image-${index}`).value;
            localStorage.setItem('products', JSON.stringify(products));
            renderProducts(); // Actualizar vista principal
        }

        function deleteProduct(index) {
            products.splice(index, 1);
            localStorage.setItem('products', JSON.stringify(products));
            renderAdminProducts();
            renderProducts();
        }

        function addNewProduct() {
            const name = document.getElementById('new-name').value;
            const price = parseFloat(document.getElementById('new-price').value);
            const image = document.getElementById('new-image').value;
            if (name && price && image) {
                const newId = products.length > 0 ? Math.max(...products.map(p => p.id)) + 1 : 1;
                products.push({ id: newId, name, price, image });
                localStorage.setItem('products', JSON.stringify(products));
                renderAdminProducts();
                renderProducts();
            }
        }

        function addDiscount() {
            const name = document.getElementById('discount-code-name').value;
            const percentage = parseFloat(document.getElementById('discount-percentage').value);
            if (name && percentage) {
                discounts.push({ name, percentage });
                localStorage.setItem('discounts', JSON.stringify(discounts));
                renderDiscounts();
            }
        }

        function renderDiscounts() {
            const discountList = document.getElementById('discount-list');
            discountList.innerHTML = '';
            discounts.forEach(discount => {
                discountList.innerHTML += `<li>${discount.name}: ${discount.percentage}% <button onclick="deleteDiscount('${discount.name}')">Eliminar</button></li>`;
            });
        }

        function deleteDiscount(name) {
            discounts = discounts.filter(d => d.name !== name);
            localStorage.setItem('discounts', JSON.stringify(discounts));
            renderDiscounts();
        }

        function closeSite() {
            localStorage.setItem('siteClosed', 'true');
            document.getElementById('maintenance').style.display = 'block';
            document.getElementById('main-content').style.display = 'none';
        }

        function openSite() {
            localStorage.setItem('siteClosed', 'false');
            document.getElementById('maintenance').style.display = 'none';
            document.getElementById('main-content').style.display = 'block';
        }

        function openWhatsApp() {
            window.open('https://wa.me/543412509449', '_blank');
        }

        // Inicializar
        renderProducts();
    </script>
</body>
</html>

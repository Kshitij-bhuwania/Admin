<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Restaurant Admin Panel</title>
    <style>
        body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; margin: 0; background: #f4f6f9; color: #2d3748; padding: 20px; display: flex; justify-content: center; }
        .admin-container { width: 100%; max-width: 800px; }
            h2, h3 { color: #1a202c; }
        .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.03); border: 1px solid #edf2f7; margin-bottom: 20px; }
        
   input, select { width: 100%; padding: 10px; margin: 8px 0 14px 0; border: 1px solid #cbd5e0; border-radius: 8px; box-sizing: border-box; font-size: 14px; outline: none; }
        input:focus, select:focus { border-color: #ff4757; }
        
   button { background: #ff4757; color: white; border: none; padding: 10px 16px; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 14px; transition: background 0.2s; }
        button:hover { background: #ff6b81; }
        
   .btn-danger { background: #e53e3e; padding: 6px 10px; font-size: 12px; }
        .btn-danger:hover { background: #c53030; }

   .list-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #edf2f7; }
        .list-item:last-child { border-bottom: none; }
        
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media(max-width: 600px) { .grid-2 { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

 <div class="admin-container">
        <h2>🛠️ Restaurant Admin Control Panel</h2>
 <!-- Manage Categories -->
        <div class="card">
            <h3>Manage Categories</h3>
            <div style="display: flex; gap: 10px;">
                <input type="text" id="newCategoryInput" placeholder="New Category Name (e.g. Desserts)" style="margin: 0;">
                <button onclick="addCategory()" style="white-space: nowrap;">Add Category</button>
            </div>
            <div id="categoryList" style="margin-top: 15px;"></div>
        </div>
        <div class="grid-2">
            <!-- Add Menu Item -->
            <div class="card">
                <h3>Add Menu Item</h3>
                <label>Item Name</label>
                <input type="text" id="itemName" placeholder="e.g. Chicken Burger">
                   <label>Price (₹)</label>
                <input type="number" id="itemPrice" placeholder="e.g. 199">
                     <label>Category</label>
                <select id="itemCategory"></select>
                         <label>Image URL</label>
             <input type="text" id="itemImage" placeholder="https://image-link.com/photo.jpg">
                
   <button onclick="addMenuItem()" style="width: 100%; margin-top: 5px;">Save Menu Item</button>
         </div>
         <!-- Existing Menu Items List with Delete -->
            <div class="card" style="max-height: 450px; overflow-y: auto;">
                <h3>Existing Menu Items</h3>
                <div id="adminMenuList"></div>
            </div>
        </div>
    </div>

<script>
    // Initialize default categories/items if not present
    if (!localStorage.getItem('categories')) {
        localStorage.setItem('categories', JSON.stringify(['Burger', 'Pizza', 'Beverages']));
    }
    if (!localStorage.getItem('menuItems')) {
        localStorage.setItem('menuItems', JSON.stringify([
            { id: '1', name: 'Cheese Burger', price: 199, category: 'Burger', image: 'https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=200' },
            { id: '2', name: 'Margherita Pizza', price: 349, category: 'Pizza', image: 'https://images.unsplash.com/photo-1604382355076-af4b0eb60143?w=200' }
        ]));
    }

    function loadAdminData() {
        renderCategories();
        renderAdminMenu();
    }

    function renderCategories() {
        let categories = JSON.parse(localStorage.getItem('categories') || '[]');
        let catListDiv = document.getElementById('categoryList');
        let dropdown = document.getElementById('itemCategory');

        catListDiv.innerHTML = '';
        dropdown.innerHTML = '';

        categories.forEach(cat => {
            // Populate category manager list
            catListDiv.innerHTML += `
                <div class="list-item">
                    <span>📁 <strong>${cat}</strong></span>
                    <button class="btn-danger" onclick="deleteCategory('${cat}')">Delete</button>
                </div>
            `;
            // Populate item form dropdown
            dropdown.innerHTML += `<option value="${cat}">${cat}</option>`;
        });
    }

    function addCategory() {
        let input = document.getElementById('newCategoryInput');
        let newCat = input.value.trim();
        if (!newCat) {
            alert('Please enter a category name.');
            return;
        }

        let categories = JSON.parse(localStorage.getItem('categories') || '[]');
        if (categories.includes(newCat)) {
            alert('Category already exists!');
            return;
        }

        categories.push(newCat);
        localStorage.setItem('categories', JSON.stringify(categories));
        input.value = '';
        loadAdminData();
    }

    function deleteCategory(catName) {
        if (!confirm(`Are you sure you want to delete category "${catName}"?`)) return;

        let categories = JSON.parse(localStorage.getItem('categories') || '[]');
        categories = categories.filter(c => c !== catName);
        localStorage.setItem('categories', JSON.stringify(categories));

        loadAdminData();
    }

    function renderAdminMenu() {
        let items = JSON.parse(localStorage.getItem('menuItems') || '[]');
        let menuListDiv = document.getElementById('adminMenuList');
        menuListDiv.innerHTML = '';

        if (items.length === 0) {
            menuListDiv.innerHTML = '<p style="color: #718096; font-size: 13px;">No menu items found.</p>';
            return;
        }

        items.forEach(item => {
            menuListDiv.innerHTML += `
                <div class="list-item">
                    <div>
                        <strong style="font-size:14px;">${item.name}</strong><br>
                        <span style="font-size:12px; color:#718096;">₹${item.price} | ${item.category}</span>
                    </div>
                    <button class="btn-danger" onclick="deleteMenuItem('${item.id}')">Delete</button>
                </div>
            `;
        });
    }

    function addMenuItem() {
        let name = document.getElementById('itemName').value.trim();
        let price = parseFloat(document.getElementById('itemPrice').value);
        let category = document.getElementById('itemCategory').value;
        let image = document.getElementById('itemImage').value.trim();

        if (!name || isNaN(price) || !category) {
            alert('Please fill out all required fields (Name, Price, Category).');
            return;
        }

        if (!image) {
            image = 'https://images.unsplash.com/photo-1546069901-ba9599a7e63c?w=200'; // Default placeholder fallback
        }

        let items = JSON.parse(localStorage.getItem('menuItems') || '[]');
        let newItem = {
            id: Date.now().toString(),
            name: name,
            price: price,
            category: category,
            image: image
        };

        items.push(newItem);
        localStorage.setItem('menuItems', JSON.stringify(items));

        // Clear form fields
        document.getElementById('itemName').value = '';
        document.getElementById('itemPrice').value = '';
        document.getElementById('itemImage').value = '';

        loadAdminData();
        alert('Menu item added successfully!');
    }

    function deleteMenuItem(id) {
        if (!confirm('Are you sure you want to delete this menu item?')) return;

        let items = JSON.parse(localStorage.getItem('menuItems') || '[]');
        items = items.filter(i => i.id !== id);
        localStorage.setItem('menuItems', JSON.stringify(items));

        renderAdminMenu();
    }

    loadAdminData();
</script>
</body>
</html>

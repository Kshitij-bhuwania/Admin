<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Admin Management Portal</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f8; padding: 30px; }
        .container { max-width: 600px; margin: 0 auto; }
        .card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.05); margin-bottom: 20px; }
        h2 { color: #333; margin-top: 0; }
        label { display: block; margin: 10px 0 5px; font-weight: bold; font-size: 14px; }
        input, select { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; }
        button { background: #2f3640; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; font-weight: bold; width: 100%; }
        button:hover { background: #718093; }
        .success-msg { color: #2ed573; font-size: 13px; margin-top: 5px; display: none; }
    </style>
</head>
<body>

    <div class="container">
        <h1>Admin Control Panel</h1>
        
        <div class="card">
            <h2>Custom Delivery Charge</h2>
            <label>Delivery Fee (₹):</label>
            <input type="number" id="deliveryInput">
            <button onclick="saveDelivery()">Update Delivery Fee</button>
            <div id="deliveryMsg" class="success-msg">Delivery charge updated successfully!</div>
        </div>

        <div class="card">
            <h2>Custom Category Dropdown</h2>
            <label>New Category Name:</label>
            <input type="text" id="categoryInput" placeholder="e.g. Desserts, Drinks">
            <button onclick="saveCategory()">Add Category</button>
            <div id="categoryMsg" class="success-msg">Category added successfully!</div>
        </div>

        <div class="card">
            <h2>Add Item with Custom Image URL</h2>
            <label>Item Name:</label>
            <input type="text" id="itemName" placeholder="e.g. Chocolate Brownie">
            <label>Price (₹):</label>
            <input type="number" id="itemPrice" placeholder="e.g. 150">
            <label>Select Category:</label>
            <select id="itemCategorySelect"></select>
            <label>Item Picture URL:</label>
            <input type="text" id="itemImage" placeholder="Paste image link">
            <button onclick="saveItem()">Publish Item to Menu</button>
            <div id="itemMsg" class="success-msg">Item added to menu catalog successfully!</div>
        </div>
    </div>

<script>
    document.getElementById('deliveryInput').value = localStorage.getItem('deliveryCharge') || '40';
    loadCategoryDropdown();

    function loadCategoryDropdown() {
        const categories = JSON.parse(localStorage.getItem('categories') || '[]');
        const select = document.getElementById('itemCategorySelect');
        select.innerHTML = '';
        categories.forEach(cat => {
            select.innerHTML += `<option value="${cat}">${cat}</option>`;
        });
    }

    function saveDelivery() {
        const fee = document.getElementById('deliveryInput').value;
        localStorage.setItem('deliveryCharge', fee);
        showMsg('deliveryMsg');
    }

    function saveCategory() {
        const catName = document.getElementById('categoryInput').value.trim();
        if (!catName) return;
        let categories = JSON.parse(localStorage.getItem('categories') || '[]');
        if (!categories.includes(catName)) {
            categories.push(catName);
            localStorage.setItem('categories', JSON.stringify(categories));
            loadCategoryDropdown();
            document.getElementById('categoryInput').value = '';
            showMsg('categoryMsg');
        } else {
            alert('Category already exists.');
        }
    }

    function saveItem() {
        const name = document.getElementById('itemName').value.trim();
        const price = parseFloat(document.getElementById('itemPrice').value);
        const category = document.getElementById('itemCategorySelect').value;
        const image = document.getElementById('itemImage').value.trim() || 'https://images.unsplash.com/photo-1546069901-ba9599a7e63c?w=200';

        if (!name || isNaN(price)) {
            alert('Please provide valid item name and price.');
            return;
        }

        let items = JSON.parse(localStorage.getItem('menuItems') || '[]');
        items.push({ id: 'ITM_' + Date.now(), name, price, category, image });
        localStorage.setItem('menuItems', JSON.stringify(items));
        
        document.getElementById('itemName').value = '';
        document.getElementById('itemPrice').value = '';
        document.getElementById('itemImage').value = '';
        showMsg('itemMsg');
    }

    function showMsg(id) {
        const el = document.getElementById(id);
        el.style.display = 'block';
        setTimeout(() => { el.style.display = 'none'; }, 2500);
    }
</script>
</body>
</html>

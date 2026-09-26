<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Menu Manager</title>
    <style>
        body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; background: #f4f6f9; padding: 20px; color: #2d3748; display: flex; justify-content: center; margin: 0; }
        .container { width: 100%; max-width: 600px; }
        .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.03); border: 1px solid #edf2f7; margin-bottom: 20px; }
        h2, h3 { margin-top: 0; color: #1a202c; }
        input, select { width: 100%; padding: 10px; margin: 8px 0 16px 0; border: 1px solid #cbd5e0; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
        .btn { background: #3182ce; color: white; border: none; padding: 10px 16px; border-radius: 6px; font-weight: 600; cursor: pointer; font-size: 14px; width: 100%; }
        .btn:hover { background: #2b6cb0; }
        .btn-danger { background: #e53e3e; padding: 6px 12px; font-size: 12px; width: auto; }
        .btn-danger:hover { background: #c53030; }
        .item-row { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #edf2f7; font-size: 14px; }
    </style>
</head>
<body>

   <div class="container">
        <h2>🛠️ Admin Menu Manager (Cloud Sync)</h2>

 <!-- Add Category Card -->
   <div class="card">
            <h3>Add New Category</h3>
            <input type="text" id="newCategoryName" placeholder="Category Name (e.g., Starters, Drinks)">
            <button class="btn" onclick="addCategory()">Add Category</button>
        </div>

        <!-- Add Item Card -->
   <div class="card">
            <h3>Add Menu Item</h3>
            <label style="font-size: 13px; font-weight: 600;">Select Category</label>
            <select id="itemCategorySelect"></select>

  <label style="font-size: 13px; font-weight: 600;">Item Name</label>
            <input type="text" id="itemName" placeholder="Item Name (e.g., Paneer Tikka)">

   <label style="font-size: 13px; font-weight: 600;">Price (₹)</label>
            <input type="number" id="itemPrice" placeholder="Price">

  <button class="btn" onclick="addItem()">Add Item to Menu</button>
        </div>

        <!-- Existing Menu List Card --
  <div class="card">
            <h3>Current Live Menu</h3>
            <div id="adminMenuList">Loading menu...</div>
        </div>
    </div>

<script>
    const FIREBASE_URL = "https://test-d34cf-default-rtdb.europe-west1.firebasedatabase.app";

    async function fetchMenuData() {
        try {
            let res = await fetch(`${FIREBASE_URL}/menu.json`);
            let data = await res.json();
            // Default structure if empty: { categories: [...] }
            if (!data || !data.categories) {
                return { categories: [] };
            }
            return data;
        } catch (e) {
            return { categories: [] };
        }
    }

    async function saveMenuData(menuData) {
        await fetch(`${FIREBASE_URL}/menu.json`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(menuData)
        });
    }

    async function loadAdminPanel() {
        let menu = await fetchMenuData();
        let select = document.getElementById('itemCategorySelect');
        let listContainer = document.getElementById('adminMenuList');

        select.innerHTML = '';
        if (menu.categories.length === 0) {
            select.innerHTML = '<option value="">Create a category first</option>';
            listContainer.innerHTML = '<p style="color:#718096; font-size:14px;">No categories or items added yet.</p>';
            return;
        }

        let catHtml = '';
        menu.categories.forEach((cat, catIndex) => {
            select.innerHTML += `<option value="${catIndex}">${cat.name}</option>`;
            
            catHtml += `<div style="margin-top: 15px; font-weight:600; color:#2b6cb0; border-bottom: 2px solid #bee3f8; padding-bottom: 4px;">
                ${cat.name} <button class="btn btn-danger" style="float:right;" onclick="deleteCategory(${catIndex})">Delete Category</button>
            </div>`;

            if (!cat.items || cat.items.length === 0) {
                catHtml += `<div style="font-size:13px; color:#a0aec0; padding: 6px 0;">No items in this category.</div>`;
            } else {
                cat.items.forEach((item, itemIndex) => {
                    catHtml += `
                        <div class="item-row">
                            <span>${item.name} - <b>₹${item.price}</b></span>
                            <button class="btn btn-danger" onclick="deleteItem(${catIndex}, ${itemIndex})">Delete</button>
                        </div>
                    `;
                });
            }
        });

        listContainer.innerHTML = catHtml;
    }

    async function addCategory() {
        let name = document.getElementById('newCategoryName').value.trim();
        if (!name) { alert('Enter a category name'); return; }

        let menu = await fetchMenuData();
        menu.categories.push({ name: name, items: [] });
        
        await saveMenuData(menu);
        document.getElementById('newCategoryName').value = '';
        loadAdminPanel();
    }

    async function addItem() {
        let catIndex = document.getElementById('itemCategorySelect').value;
        let name = document.getElementById('itemName').value.trim();
        let price = parseFloat(document.getElementById('itemPrice').value);

        if (catIndex === "" || !name || isNaN(price)) {
            alert('Please fill out all item details properly.');
            return;
        }

        let menu = await fetchMenuData();
        if (!menu.categories[catIndex].items) {
            menu.categories[catIndex].items = [];
        }

        menu.categories[catIndex].items.push({ name: name, price: price });
        
        await saveMenuData(menu);
        document.getElementById('itemName').value = '';
        document.getElementById('itemPrice').value = '';
        loadAdminPanel();
    }

    async function deleteCategory(catIndex) {
        if (!confirm('Delete this category and all its items?')) return;
        let menu = await fetchMenuData();
        menu.categories.splice(catIndex, 1);
        await saveMenuData(menu);
        loadAdminPanel();
    }

    async function deleteItem(catIndex, itemIndex) {
        if (!confirm('Delete this item?')) return;
        let menu = await fetchMenuData();
        menu.categories[catIndex].items.splice(itemIndex, 1);
        await saveMenuData(menu);
        loadAdminPanel();
    }

    loadAdminPanel();
</script>
</body>
</html>

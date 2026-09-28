<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Dashboard</title>
    <style>
        body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; background: #f4f6f9; padding: 20px; color: #2d3748; display: flex; justify-content: center; margin: 0; }
        .container { width: 100%; max-width: 600px; }
        .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.03); border: 1px solid #edf2f7; margin-bottom: 20px; }
        h2, h3 { margin-top: 0; color: #1a202c; }
        input, select { width: 100%; padding: 10px; margin: 8px 0 16px 0; border: 1px solid #cbd5e0; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
        .btn { background: #3182ce; color: white; border: none; padding: 10px 16px; border-radius: 6px; font-weight: 600; cursor: pointer; font-size: 14px; width: 100%; transition: background 0.2s; }
        .btn:hover { background: #2b6cb0; }
        .btn:disabled { background: #a0aec0; cursor: not-allowed; }
        .btn-danger { background: #e53e3e; padding: 6px 12px; font-size: 12px; width: auto; }
        .btn-danger:hover { background: #c53030; }
        .btn-warning { background: #d69e2e; padding: 6px 12px; font-size: 12px; width: auto; color: white; margin-right: 5px; }
        .btn-warning:hover { background: #b7791f; }
        .item-row { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #edf2f7; font-size: 14px; }
        .status-msg { padding: 8px; border-radius: 6px; font-size: 13px; text-align: center; margin-bottom: 12px; display: none; }
        .success { background: #c6f6d5; color: #22543d; }
        .image-preview { width: 60px; height: 60px; object-fit: cover; border-radius: 6px; border: 1px solid #cbd5e0; display: none; margin-bottom: 12px; }
    </style>
</head>
<body>

   <div class="container">
        <h2>🛠️ Admin Dashboard</h2>

        <!-- Delivery Pricing Settings Card -->
        <div class="card">
            <h3>Delivery Pricing Rules (Distance Based)</h3>
            <div id="deliveryStatus" class="status-msg success">Settings saved successfully!</div>
           
            <label style="font-size: 13px; font-weight: 600;">Base Distance Limit (km)</label>
            <input type="number" id="baseKm" value="3" step="0.5">

            <label style="font-size: 13px; font-weight: 600;">Base Price for Base Distance (₹)</label>
            <input type="number" id="basePrice" value="30">

            <label style="font-size: 13px; font-weight: 600;">Additional Price per Extra km (₹)</label>
            <input type="number" id="extraPricePerKm" value="15">

            <button class="btn" id="saveDeliveryBtn" onclick="saveDeliverySettings()">Save Delivery Pricing</button>
        </div>

        <!-- Add Category Card -->
        <div class="card">
            <h3>Add New Category</h3>
            <input type="text" id="newCategoryName" placeholder="Category Name (e.g., Starters, Drinks)">
            <button class="btn" id="addCategoryBtn" onclick="addCategory()">Add Category</button>
        </div>

        <!-- Add / Edit Item Card -->
        <div class="card">
            <h3 id="itemCardTitle">Add Menu Item</h3>
            <input type="hidden" id="editCatIndex" value="">
            <input type="hidden" id="editItemIndex" value="">

            <label style="font-size: 13px; font-weight: 600;">Select Category</label>
            <select id="itemCategorySelect"></select>

            <label style="font-size: 13px; font-weight: 600;">Item Name</label>
            <input type="text" id="itemName" placeholder="Item Name (e.g., Paneer Tikka)">

            <label style="font-size: 13px; font-weight: 600;">Price (₹)</label>
            <input type="number" id="itemPrice" placeholder="Price">

            <label style="font-size: 13px; font-weight: 600;">Upload Dish Image</label>
            <input type="file" id="itemImageFile" accept="image/*" onchange="handleImageUpload(event)">
            <img id="imgPreview" class="image-preview" alt="Preview">

            <button class="btn" id="saveItemBtn" onclick="saveItem()">Add Item to Menu</button>
            <button class="btn" id="cancelEditBtn" onclick="resetItemForm()" style="background: #718096; margin-top: 8px; display: none;">Cancel Edit</button>
        </div>

        <!-- Existing Menu List Card -->
        <div class="card">
            <h3>Current Live Menu</h3>
            <div id="adminMenuList">Loading menu...</div>
        </div>
    </div>

<script>
    const FIREBASE_URL = "https://test-d34cf-default-rtdb.europe-west1.firebasedatabase.app";
    let uploadedImageBase64 = "";

    function handleImageUpload(event) {
        let file = event.target.files[0];
        if (!file) return;

        let reader = new FileReader();
        reader.onload = function(e) {
            let img = new Image();
            img.onload = function() {
                let canvas = document.createElement('canvas');
                let ctx = canvas.getContext('2d');
                let MAX_WIDTH = 400;
                let MAX_HEIGHT = 400;
                let width = img.width;
                let height = img.height;

                if (width > height) {
                    if (width > MAX_WIDTH) {
                        height *= MAX_WIDTH / width;
                        width = MAX_WIDTH;
                    }
                } else {
                    if (height > MAX_HEIGHT) {
                        width *= MAX_HEIGHT / height;
                        height = MAX_HEIGHT;
                    }
                }

                canvas.width = width;
                canvas.height = height;
                ctx.drawImage(img, 0, 0, width, height);
                
                uploadedImageBase64 = canvas.toDataURL('image/jpeg', 0.7); // 70% quality compression
                
                let preview = document.getElementById('imgPreview');
                preview.src = uploadedImageBase64;
                preview.style.display = 'block';
            }
            img.src = e.target.result;
        }
        reader.readAsDataURL(file);
    }

    async function loadDeliverySettings() {
        try {
            let res = await fetch(`${FIREBASE_URL}/settings/delivery.json`);
            let data = await res.json();
            if (data) {
                document.getElementById('baseKm').value = data.baseKm ?? 3;
                document.getElementById('basePrice').value = data.basePrice ?? 30;
                document.getElementById('extraPricePerKm').value = data.extraPricePerKm ?? 15;
            }
        } catch (e) {
            console.log("Could not load delivery settings");
        }
    }

    async function saveDeliverySettings() {
        let btn = document.getElementById('saveDeliveryBtn');
        btn.disabled = true;
        btn.innerText = "Saving...";

        let settings = {
            baseKm: parseFloat(document.getElementById('baseKm').value),
            basePrice: parseInt(document.getElementById('basePrice').value),
            extraPricePerKm: parseInt(document.getElementById('extraPricePerKm').value)
        };

        try {
            await fetch(`${FIREBASE_URL}/settings/delivery.json`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(settings)
            });
            let msg = document.getElementById('deliveryStatus');
            msg.style.display = 'block';
            setTimeout(() => { msg.style.display = 'none'; }, 3000);
        } catch (e) {
            alert('Failed to save delivery settings.');
        } finally {
            btn.disabled = false;
            btn.innerText = "Save Delivery Pricing";
        }
    }

    async function fetchMenuData() {
        try {
            let res = await fetch(`${FIREBASE_URL}/menu.json`);
            let data = await res.json();
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
        await loadDeliverySettings();
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
                    let imgThumb = item.image ? `<img src="${item.image}" style="width: 28px; height: 28px; object-fit: cover; border-radius: 4px; vertical-align: middle; margin-right: 8px;">` : '';
                    catHtml += `
                        <div class="item-row">
                            <span style="display: flex; align-items: center; max-width: 60%;">
                                ${imgThumb}
                                <span style="overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">${item.name} - <b>₹${item.price}</b></span>
                            </span>
                            <div>
                                <button class="btn btn-warning" onclick="editItem(${catIndex}, ${itemIndex})">Edit</button>
                                <button class="btn btn-danger" onclick="deleteItem(${catIndex}, ${itemIndex})">Delete</button>
                            </div>
                        </div>
                    `;
                });
            }
        });

        listContainer.innerHTML = catHtml;
    }

    async function addCategory() {
        let nameField = document.getElementById('newCategoryName');
        let name = nameField.value.trim();
        if (!name) { alert('Enter a category name'); return; }

        let menu = await fetchMenuData();
        menu.categories.push({ name: name, items: [] });
        
        await saveMenuData(menu);
        nameField.value = '';
        loadAdminPanel();
    }

    async function saveItem() {
        let catIndex = document.getElementById('itemCategorySelect').value;
        let name = document.getElementById('itemName').value.trim();
        let price = parseFloat(document.getElementById('itemPrice').value);
        let editCat = document.getElementById('editCatIndex').value;
        let editItem = document.getElementById('editItemIndex').value;

        if (catIndex === "" || !name || isNaN(price)) {
            alert('Please fill out all item details properly.');
            return;
        }

        let menu = await fetchMenuData();

        let finalImage = uploadedImageBase64;
        if (!finalImage && editCat !== "" && editItem !== "") {
            finalImage = menu.categories[parseInt(editCat)].items[parseInt(editItem)].image || "";
        }

        if (editCat !== "" && editItem !== "") {
            let oldCatIndex = parseInt(editCat);
            let oldItemIndex = parseInt(editItem);

            if (oldCatIndex !== parseInt(catIndex)) {
                menu.categories[oldCatIndex].items.splice(oldItemIndex, 1);
                if (!menu.categories[catIndex].items) menu.categories[catIndex].items = [];
                menu.categories[catIndex].items.push({ name, price, image: finalImage });
            } else {
                menu.categories[catIndex].items[oldItemIndex] = { name, price, image: finalImage };
            }
        } else {
            if (!menu.categories[catIndex].items) {
                menu.categories[catIndex].items = [];
            }
            menu.categories[catIndex].items.push({ name, price, image: finalImage });
        }
        
        await saveMenuData(menu);
        resetItemForm();
        loadAdminPanel();
    }

    async function editItem(catIndex, itemIndex) {
        let menu = await fetchMenuData();
        let item = menu.categories[catIndex].items[itemIndex];
        
        document.getElementById('itemCategorySelect').value = catIndex;
        document.getElementById('itemName').value = item.name;
        document.getElementById('itemPrice').value = item.price;
        
        uploadedImageBase64 = item.image || "";
        let preview = document.getElementById('imgPreview');
        if (uploadedImageBase64) {
            preview.src = uploadedImageBase64;
            preview.style.display = 'block';
        } else {
            preview.style.display = 'none';
        }

        document.getElementById('editCatIndex').value = catIndex;
        document.getElementById('editItemIndex').value = itemIndex;

        document.getElementById('itemCardTitle').innerText = "Edit Menu Item";
        document.getElementById('saveItemBtn').innerText = "Update Menu Item";
        document.getElementById('cancelEditBtn').style.display = 'block';

        window.scrollTo({ top: 400, behavior: 'smooth' });
    }

    function resetItemForm() {
        document.getElementById('itemName').value = '';
        document.getElementById('itemPrice').value = '';
        document.getElementById('itemImageFile').value = '';
        document.getElementById('editCatIndex').value = '';
        document.getElementById('editItemIndex').value = '';
        document.getElementById('imgPreview').style.display = 'none';
        uploadedImageBase64 = "";

        document.getElementById('itemCardTitle').innerText = "Add Menu Item";
        document.getElementById('saveItemBtn').innerText = "Add Item to Menu";
        document.getElementById('cancelEditBtn').style.display = 'none';
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

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin - Delivery Pricing Settings</title>
    <style>
        body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; background: #f4f6f9; padding: 20px; display: flex; justify-content: center; align-items: center; min-height: 100vh; box-sizing: border-box; color: #2d3748; margin: 0; }
        .admin-box { background: white; padding: 30px 20px; border-radius: 16px; width: 100%; max-width: 450px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); border: 1px solid #edf2f7; box-sizing: border-box; }
        h2 { margin-top: 0; color: #1a202c; font-weight: 600; font-size: 22px; text-align: center; }
        .form-group { margin-bottom: 15px; text-align: left; }
        label { display: block; font-size: 13px; font-weight: 600; color: #4a5568; margin-bottom: 5px; }
        input { width: 100%; padding: 10px; border: 1px solid #cbd5e0; border-radius: 8px; font-size: 14px; box-sizing: border-box; }
        .btn { background: #3182ce; color: white; border: none; padding: 12px; border-radius: 8px; width: 100%; font-weight: 600; cursor: pointer; font-size: 14px; margin-top: 10px; transition: background 0.2s; }
        .btn:hover { background: #2b6cb0; }
        .btn-back { background: #718096; margin-top: 8px; }
        .btn-back:hover { background: #4a5568; }
        .status-msg { padding: 10px; border-radius: 8px; font-size: 13px; text-align: center; margin-bottom: 15px; display: none; }
        .success { background: #c6f6d5; color: #22543d; }
        .loading { background: #ebf8ff; color: #2b6cb0; }
    </style>
</head>
<body>

   <div class="admin-box">
        <h2>⚙️ Delivery Pricing Admin</h2>
        <p style="font-size: 13px; color: #718096; margin-bottom: 20px; text-align: center;">Manage delivery distance limits and fees for Faven Lightings (Synced Live via Firebase).</p>

  <div id="statusMsg" class="status-msg"></div>

   <div class="form-group">
            <label>Tier 1 Max Distance (km):</label>
            <input type="number" id="tier1Dist" value="3" step="0.5">
        </div>
        <div class="form-group">
            <label>Tier 1 Delivery Charge (₹):</label>
            <input type="number" id="tier1Price" value="30">
        </div>

   <hr style="border:0; border-top:1px dashed #cbd5e0; margin:15px 0;">

  <div class="form-group">
            <label>Tier 2 Max Distance (km):</label>
            <input type="number" id="tier2Dist" value="6" step="0.5">
        </div>
        <div class="form-group">
            <label>Tier 2 Delivery Charge (₹):</label>
            <input type="number" id="tier2Price" value="60">
        </div>

  <hr style="border:0; border-top:1px dashed #cbd5e0; margin:15px 0;">

   <div class="form-group">
            <label>Above Tier 2 Delivery Charge (₹):</label>
            <input type="number" id="farPrice" value="100">
        </div>

   <button class="btn" onclick="saveSettings()">💾 Save & Sync Pricing</button>
        <button class="btn btn-back" onclick="goToKitchen()">← Back to Kitchen Orders</button>
    </div>

<script>
    const FIREBASE_URL = "https://test-d34cf-default-rtdb.europe-west1.firebasedatabase.app";

    // Load settings from Firebase when admin page opens
    async function loadSettings() {
        showStatus("Loading current settings from Firebase...", "loading");
        try {
            let res = await fetch(`${FIREBASE_URL}/settings/delivery.json`);
            let data = await res.json();
            if (data) {
                document.getElementById('tier1Dist').value = data.tier1Dist ?? 3;
                document.getElementById('tier1Price').value = data.tier1Price ?? 30;
                document.getElementById('tier2Dist').value = data.tier2Dist ?? 6;
                document.getElementById('tier2Price').value = data.tier2Price ?? 60;
                document.getElementById('farPrice').value = data.farPrice ?? 100;
            }
            hideStatus();
        } catch (e) {
            showStatus("Failed to load settings from server.", "success");
        }
    }

    // Save updated settings to Firebase
    async function saveSettings() {
        const settings = {
            tier1Dist: parseFloat(document.getElementById('tier1Dist').value),
            tier1Price: parseInt(document.getElementById('tier1Price').value),
            tier2Dist: parseFloat(document.getElementById('tier2Dist').value),
            tier2Price: parseInt(document.getElementById('tier2Price').value),
            farPrice: parseInt(document.getElementById('farPrice').value)
        };

        showStatus("Saving settings to Firebase...", "loading");
        try {
            await fetch(`${FIREBASE_URL}/settings/delivery.json`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(settings)
            });
            showStatus("✅ Settings saved and synced successfully!", "success");
            setTimeout(hideStatus, 3000);
        } catch (e) {
            alert('Failed to save settings to Firebase.');
            hideStatus();
        }
    }

    function showStatus(text, type) {
        const msg = document.getElementById('statusMsg');
        msg.className = 'status-msg ' + type;
        msg.innerText = text;
        msg.style.display = 'block';
    }

    function hideStatus() {
        document.getElementById('statusMsg').style.display = 'none';
    }

    function goToKitchen() {
        window.location.href = 'orders.html';
    }

    loadSettings();
</script>
</body>
</html>

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Figurine Store Dashboard</title>
    <style>a
        body {
            font-family: Arial, sans-serif;
            background-color: #ffebee; /* Coral background color */
            margin: 0;
            padding: 20px;
        }
        h1 {
            text-align: center;
            color: #333;
        }
        .dashboard {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
        }
        .figurine-box {
            background-color: #ffccbc;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
            margin: 10px;
            padding: 15px;
            width: 250px;
            text-align: center;
        }
        .figurine-name {
            font-size: 1.2em;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .figurine-details {
            margin-bottom: 10px;
        }
        .figurine-price {
            color: #28a745;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .figurine-quantity {
            color: #007bff;
        }
    </style>
</head>
<body>

    <h1>Figurine Store Dashboard</h1>
    <div class="dashboard">
        <!-- Figurine 1 -->
        <div class="figurine-box">
            <div class="figurine-name">Dragon Warrior</div>
            <div class="figurine-details">ID: 001</div>
            <div class="figurine-details">Description: A fierce dragon warrior figurine.</div>
            <div class="figurine-price">$29.99</div>
            <div class="figurine-quantity">Quantity: 10</div>
        </div>

        <!-- Figurine 2 -->
        <div class="figurine-box">
            <div class="figurine-name">Magic Unicorn</div>
            <div class="figurine-details">ID: 002</div>
            <div class="figurine-details">Description: A beautiful unicorn figurine with a glittering mane.</div>
            <div class="figurine-price">$24.99</div>
            <div class="figurine-quantity">Quantity: 15</div>
        </div>

        <!-- Figurine 3 -->
        <div class="figurine-box">
            <div class="figurine-name">Knight in Armor</div>
            <div class="figurine-details">ID: 003</div>
            <div class="figurine-details">Description: A brave knight ready for battle.</div>
            <div class="figurine-price">$34.99</div>
            <div class="figurine-quantity">Quantity: 5</div>
        </div>

        <!-- Figurine 4 -->
        <div class="figurine-box">
            <div class="figurine-name">Mermaid Princess</div>
            <div class="figurine-details">ID: 004</div>
            <div class="figurine-details">Description: A lovely mermaid figurine with a shimmering tail.</div>
            <div class="figurine-price">$19.99</div>
            <div class="figurine-quantity">Quantity: 20</div>
        </div>
    </div>

</body>
</html>
```

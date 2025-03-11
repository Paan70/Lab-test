<?php
session_start();
// Initialize session if not exists
if (!isset($_SESSION['bookings'])) {
    $_SESSION['bookings'] = [];
}

// Destination data
$destinations = [
    'Paris' => 500,
    'Tokyo' => 800,
    'London' => 600,
    'Dubai' => 400,
    'Sydney' => 700
];

// Accommodation rates
$accommodationRates = [
    'budget' => 50,
    'standard' => 100,
    'luxury' => 200
];

function calculateTotalCost($destination, $numTravelers, $numDays, $accommodationType) {
    global $destinations, $accommodationRates;
    
    // Base calculations
    $flightCost = $destinations[$destination] * $numTravelers;
    $accommodationCost = $accommodationRates[$accommodationType] * $numTravelers * $numDays;
    $dailyExpenses = 50 * $numTravelers * $numDays;
    
    $subtotal = $flightCost + $accommodationCost + $dailyExpenses;
    
    // Apply discount for groups
    $discount = ($numTravelers > 4) ? $subtotal * 0.1 : 0;
    
    return [
        'flight' => $flightCost,
        'accommodation' => $accommodationCost,
        'daily' => $dailyExpenses,
        'subtotal' => $subtotal,
        'discount' => $discount,
        'total' => $subtotal - $discount
    ];
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_POST['delete'])) {
        // Handle deletion
        $index = (int)$_POST['index'];
        if (isset($_SESSION['bookings'][$index])) {
            array_splice($_SESSION['bookings'], $index, 1);
        }
    } else {
        // Handle new booking
        $booking = [
            'name' => $_POST['name'],
            'destination' => $_POST['destination'],
            'travelers' => (int)$_POST['travelers'],
            'days' => (int)$_POST['days'],
            'accommodation' => $_POST['accommodation'],
            'cost' => calculateTotalCost(
                $_POST['destination'],
                (int)$_POST['travelers'],
                (int)$_POST['days'],
                $_POST['accommodation']
            )
        ];
        
        $_SESSION['bookings'][] = $booking;
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vacation Budget Calculator</title>
    <style>
        .container { max-width: 800px; margin: 20px auto; padding: 20px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { padding: 10px; border: 1px solid #ddd; text-align: left; }
        th { background-color: #f5f5f5; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Vacation Budget Calculator</h2>
        
        <form method="post">
            <div>
                <label>Traveller's Name:</label>
                <input type="text" name="name" required>
            </div>
            
            <div>
                <label>Destination:</label>
                <select name="destination" required>
                    <?php foreach ($destinations as $city => $price): ?>
                        <option value="<?= $city ?>">
                            <?= $city ?> (RM<?= number_format($price) ?>)
                        </option>
                    <?php endforeach; ?>
                </select>
            </div>
            
            <div>
                <label>Number of Travellers:</label>
                <input type="number" name="travelers" min="1" required>
            </div>
            
            <div>
                <label>Number of Days:</label>
                <input type="number" name="days" min="1" required>
            </div>
            
            <div>
                <label>Accommodation Type:</label>
                <select name="accommodation" required>
                    <option value="budget">Budget (RM50/day)</option>
                    <option value="standard">Standard (RM100/day)</option>
                    <option value="luxury">Luxury (RM200/day)</option>
                </select>
            </div>
            
            <button type="submit">Calculate</button>
        </form>

        <?php if (!empty($_SESSION['bookings'])): ?>
            <h3>Booking Records</h3>
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Destination</th>
                        <th>Travellers</th>
                        <th>Days</th>
                        <th>Accommodation</th>
                        <th>Total Cost</th>
                        <th>Action</th>
                    </tr>
                </thead>
                <tbody>
                    <?php foreach ($_SESSION['bookings'] as $index => $booking): ?>
                        <tr>
                            <td><?= htmlspecialchars($booking['name']) ?></td>
                            <td><?= $booking['destination'] ?></td>
                            <td><?= $booking['travelers'] ?></td>
                            <td><?= $booking['days'] ?></td>
                            <td><?= ucfirst($booking['accommodation']) ?></td>
                            <td>
                                RM<?= number_format($booking['cost']['total'], 2) ?>
                                <?php if ($booking['cost']['discount'] > 0): ?>
                                    <br><small>(10% discount applied)</small>
                                <?php endif; ?>
                            </td>
                            <td>
                                <form method="post">
                                    <input type="hidden" name="index" value="<?= $index ?>">
                                    <button type="submit" name="delete">Delete</button>
                                </form>
                            </td>
                        </tr>
                    <?php endforeach; ?>
                </tbody>
            </table>
        <?php endif; ?>
    </div>
</body>
</html>

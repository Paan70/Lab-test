<?php
// Define destination
$destination = [
    "Paris " => 500,
    "Tokyo " => 800,
    "London " => 600,
    "Dubai " => 400,
];

//Define daily expenses
$dailyExpenses = [
    "Perperson" => 50,
];

// Define accommodation
$accommodation = [
    "budget" => 50,  // per night per person
    "Standard" => 100, 
    "Luxury" => 200, 
];

// Start session to store customer data
session_start();
if (!isset($_SESSION['customers'])) {
    $_SESSION['customers'] = [];
}

// Function to calculate total fare
function calculateFare($destination, $accommodation, $numTraveller, $dailyExpenses, $numberOfDays) {
    global $routes, $fare_rates;
    
    if (!isset($destinations[$destination]) || !isset($fare_rates[$ticketType])) {
        return "Invalid route or ticket type.";
    }
    
    $destination = $destinations[$destination];
    $rate = $accommodations[$accommodation];
    $totalFare = $destination * $rate * $numPassengers;
    
    return number_format($totalFare, 2);
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (isset($_POST['delete']) && isset($_POST['index'])) {
        // Delete customer entry
        $index = (int)$_POST['index'];
        if (isset($_SESSION['customers'][$index])) {
            array_splice($_SESSION['customers'], $index, 1);
        }
    } else {
        // Add new customer entry
        $name = $_POST['name'];
        $destination = $_POST['destination'];
        $numPassengers = (int)$_POST['numPassengers'];
        $accommodation = $_POST['accommodation'];
        $dailyExpenses = $_POST['dailyExpenses'];
        $numberOfDays = (int)$_POST['numberOfDays'];
        
        $fare = calculateFare($destination, $accommodation, $numPassengers, $dailyExpenses, $numberOfDays);
        
        $_SESSION['customers'][] = [
            'name' => $name,
            'destination' => $destination,
            'numPassengers' => $numPassengers,
            'accommodation' => $accommodation,
            'dailyExpenses' => $dailyExpenses,
            'numberOfDays' => $numberOfDays,
            'fare' => $fare
        ];
    }
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>Vacation Budget Calculator</title>
</head>
<body>
    <h2>Online Vacation Budget Calculator</h2>
    <form method="post">
        Name: <input type="text" name="name" required><br>
        Destination:
        <select name="destination" required>
            <?php foreach ($destination as $key => $value) { echo "<option value='$key'>$key</option>"; } ?>
        </select><br>
        Number of Passengers: <input type="number" name="numPassengers" min="1" required><br>
        Number of Days: <input type="number" name="numOfdays" min="1" required><br>
        Ticket Type:
        <select name="accommodation" required>
            <option value="budget">Budget</option>
            <option value="standard">Standard</option>
            <option value="luxury">Luxury</option>
        </select><br>
        <input type="submit" value="Calculate Fare">
    </form>
    
    <?php if (!empty($_SESSION['customers'])): ?>
        <h3>Fare Details</h3>
        <table border="1">
            <tr><th>Name</th><th>Destination</th><th>Passengers</th><th>Number of days<th>Accommodation Type</th><th>Total Fare (RM)</th><th>Action</th></tr>
            <?php foreach ($_SESSION['customers'] as $index => $customer): ?>
                <tr>
                    <td><?php echo htmlspecialchars($customer['name']); ?></td>
                    <td><?php echo htmlspecialchars($customer['destination']); ?></td>
                    <td><?php echo htmlspecialchars($customer['numPassengers']); ?></td>
                     <td><?php echo htmlspecialchars($customer['numberOfDays']); ?></td>
                    <td><?php echo htmlspecialchars($customer['accommodation']); ?></td>
                    <td><?php echo htmlspecialchars($customer['fare']); ?></td>
    
                    <td>
                        <form method="post" style="display:inline;">
                            <input type="hidden" name="index" value="<?php echo $index; ?>">
                            <input type="submit" name="delete" value="Delete">
                        </form>
                    </td>
                </tr>
            <?php endforeach; ?>
        </table>
    <?php endif; ?>
</body>
</html>

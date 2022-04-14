## Create an HTML version
You can create an HTML version of the main documents. To do so, 
1. Uncomment the first line on the main readme.md file `<!-- <script src="https://kit.fontawesome.com/81ab11011e.js" crossorigin="anonymous"></script> -->` . This script will load the icons into the HTML. 
1. Convert the MD file into HTML using [pandoc](https://pandoc.org/), like so:

    ```console
    pandoc --toc --metadata title="ITM Platform's Plugin Script Documentation" --standalone --from gfm --to html5 --css style.css ../readme.md -o index.html 
    ```
1. Comment back the script line in readme.md to avoid showing it on gitHub.



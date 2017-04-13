# About

Pixi-tilemap allows a developer to create tilemaps based on PIXI.js and https://github.com/pixijs/pixi-tilemap

# Example Usage

Completed demo: https://alanhollis.com/pixi-tilemap-tutorial/

## Creating a basic tile map

```javascript
<!doctype html>
<html>
<head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/4.4.3/pixi.min.js"></script>
    <script src="https://cdn.rawgit.com/pixijs/pixi-tilemap/master/bin/pixi-tilemap.js"></script>
</head>
<body>

<script>
    var resolutionX = 800;
    var resolutionY = 600;
    var tileSizeX = 128;
    var tileSizeY = 128;

    var app = new PIXI.Application(resolutionX, resolutionY);
    document.body.appendChild(app.view);

    PIXI.loader
            .add([
                "imgs/imgGround.png"
            ])
            .load(setup);

    function setup() {

        var groundTiles = new PIXI.tilemap.CompositeRectTileLayer(0, PIXI.utils.TextureCache['imgs/imgGround.png']);
        app.stage.addChild(groundTiles);


        for (var i = 0; i <= parseInt(resolutionX / tileSizeX); i++) {
            for (var j = 0; j <= parseInt(resolutionX / tileSizeX); j++) {
                groundTiles.addFrame('imgs/imgGround.png', i * tileSizeX, j * tileSizeY);
            }
        }
    }
</script>
</body>
</html>
```

## Pivoting the tile map around a player to give a movement effect

```javascript
<!doctype html>
<html>
<head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/4.4.3/pixi.min.js"></script>
    <script src="https://cdn.rawgit.com/pixijs/pixi-tilemap/master/bin/pixi-tilemap.js"></script>
</head>
<body>

<script>
    var resolutionX = 800;
    var resolutionY = 600;
    var tileSizeX = 128;
    var tileSizeY = 128;

    var app = new PIXI.Application(resolutionX, resolutionY);
    document.body.appendChild(app.view);

    var groundTiles;
    var playerTankSprite;
    var playerOffsetX = (resolutionX / 2 - 24);
    var playerOffsetY = (resolutionY / 2 - 24);

    var player = {
        x: 0,
        y: 0
    };

    PIXI.loader
            .add([
                "imgs/imgGround.png",
                "imgs/imgTanks.png"
            ])
            .load(setup);

    function setup() {

        // Create our tile map based on the ground texture
        groundTiles = new PIXI.tilemap.CompositeRectTileLayer(0, PIXI.utils.TextureCache['imgs/imgGround.png']);
        groundTiles.position.set(playerOffsetX + player.x, playerOffsetY + player.y);
        drawTiles();

        app.stage.addChild(groundTiles);

        // Place a tank in the middle of the texture the tank sprite never moves
        // Instead we move the tile map around the player in out game loop
        var tankTexture = new PIXI.Texture(
                PIXI.utils.TextureCache['imgs/imgTanks.png'],
                new PIXI.Rectangle(0 * 48, 0, 48, 48)
        );

        playerTankSprite = new PIXI.Sprite(tankTexture);
        playerTankSprite.x = playerOffsetX;
        playerTankSprite.y = playerOffsetY;
        app.stage.addChild(playerTankSprite);

        gameLoop();
    }

    function drawTiles() {
        // The +5 gives us a buffer around the current player
        var numberOfTiles = parseInt(resolutionX / tileSizeX) + 5;

        for (var i = -numberOfTiles; i <= numberOfTiles; i++) {
            for (var j = -numberOfTiles; j <= numberOfTiles; j++) {
                groundTiles.addFrame('imgs/imgGround.png', i * tileSizeX, j * tileSizeY);
            }
        }
    }

    function gameLoop() {

        // Make it look like the tank is driving forwards by moving the tiles
        player.y -= 4;
        groundTiles.pivot.set(player.x, player.y);
        requestAnimationFrame(gameLoop);
    }

</script>
</body>
</html>
```

## Redrawing the tile map to give  the illusion of constant movement

Change the draw tiles function and the gampe loop function as follows


```javascript
    function drawTiles() {
        var numberOfTiles = parseInt(resolutionX / tileSizeX) + 10;

        // We need to calculate this in order to prevent the tile looking "jumpy" when it's redrawn
        
        var groundOffsetX = player.x % 128; // Number of tank tiles on x axis
        var groundOffsetY = player.y % 128; // Number of tank tiles on y axis

        for (var i = -numberOfTiles; i <= numberOfTiles; i++) {
            for (var j = -numberOfTiles; j <= numberOfTiles; j++) {
                groundTiles.addFrame('imgs/imgGround.png', i * tileSizeX, j * tileSizeY);
            }
        }

        groundTiles.position.set(playerOffsetX + player.x - groundOffsetX, playerOffsetY + player.y - groundOffsetY);
    }

    function gameLoop() {

        if (player.y % (10 * tileSizeY ) === -0) {
            drawTiles();
        }

        // Make it look like the tank is driving forwards by moving the tiles
        player.y -= 4;
        groundTiles.pivot.set(player.x, player.y);
        requestAnimationFrame(gameLoop);
    }
```

## Add another image to the tile map

```javascript

    function drawTiles() {
        var numberOfTiles = parseInt(resolutionX / tileSizeX) + 10;

        // We need to calculate this in order to prevent the tile looking "jumpy" when it's redrawn

        var groundOffsetX = player.x % 128; // Number of tank tiles on x axis
        var groundOffsetY = player.y % 128; // Number of tank tiles on y axis

        for (var i = -numberOfTiles; i <= numberOfTiles; i++) {
            for (var j = -numberOfTiles; j <= numberOfTiles; j++) {
                groundTiles.addFrame('imgs/imgGround.png', i * tileSizeX, j * tileSizeY);
            }
        }



        // We'll use these later to animate the building
        var animateX = 1;
        var animateY = 0;
        
        /**
         * We'll draw the building off the players viewport to start to give it the impression we're driving past
         * @type {number}
         */
        var buildingTexture = new PIXI.Texture(
                PIXI.utils.TextureCache['imgs/imgBuildings.png'],
                new PIXI.Rectangle(0, 0, 144, 144, 144)
        );
        groundTiles.addFrame(buildingTexture, (2 * 128), (4 * 128) * -1, animateX, animateY);
        groundTiles.position.set(playerOffsetX + player.x - groundOffsetX, playerOffsetY + player.y - groundOffsetY);
    }
```

## Add animation to the building


```javascript
    var tick = new Date().getTime();
    var tileAnim = 0;
    var tileAnimationTick = 0;
    function gameLoop() {


        tick = new Date().getTime();
        if (player.y % (10 * tileSizeY ) === -0) {
            console.log("redrawing");
            drawTiles();
        }

        // Make it look like the tank is driving forwards by moving the tiles
        player.y -= 4;
        groundTiles.pivot.set(player.x, player.y);

        app.renderer.plugins.tilemap.tileAnim[0] = tileAnim * 144;
        if (tick > tileAnimationTick) {
            tileAnimationTick = tick + 300;

            tileAnim = tileAnim + 1;
            if (tileAnim >= 3) {

                tileAnim = 0;
            }
        }
        requestAnimationFrame(gameLoop);
    }
```
    
## Complete code with stats

```javascript
<!doctype html>
<html>
<head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/4.4.3/pixi.min.js"></script>
    <script src="https://cdn.rawgit.com/pixijs/pixi-tilemap/master/bin/pixi-tilemap.js"></script>
    <script src="//rawgit.com/mrdoob/stats.js/master/build/stats.min.js"></script>
</head>
<body>

<script>
    var resolutionX = 800;
    var resolutionY = 600;
    var tileSizeX = 128;
    var tileSizeY = 128;

    var stats = new Stats();
    stats.showPanel(0);
    document.body.appendChild(stats.dom);

    var app = new PIXI.Application(resolutionX, resolutionY);
    document.body.appendChild(app.view);

    var groundTiles;
    var playerTankSprite;
    var playerOffsetX = (resolutionX / 2 - 24);
    var playerOffsetY = (resolutionY / 2 - 24);

    var player = {
        x: 0,
        y: 0
    };

    PIXI.loader
            .add([
                "imgs/imgGround.png",
                "imgs/imgTanks.png",
                "imgs/imgBuildings.png"
            ])
            .load(setup);

    function setup() {

        // Create our tile map based on the ground texture
        groundTiles = new PIXI.tilemap.CompositeRectTileLayer(0, PIXI.utils.TextureCache['imgs/imgGround.png']);
        drawTiles();

        app.stage.addChild(groundTiles);

        // Place a tank in the middle of the texture the tank sprite never moves
        // Instead we move the tile map around the player in out game loop
        var tankTexture = new PIXI.Texture(
                PIXI.utils.TextureCache['imgs/imgTanks.png'],
                new PIXI.Rectangle(0 * 48, 0, 48, 48)
        );

        playerTankSprite = new PIXI.Sprite(tankTexture);
        playerTankSprite.x = playerOffsetX;
        playerTankSprite.y = playerOffsetY;
        app.stage.addChild(playerTankSprite);

        gameLoop();
    }

    function drawTiles() {
        var numberOfTiles = parseInt(resolutionX / tileSizeX) + 10;

        // We need to calculate this in order to prevent the tile looking "jumpy" when it's redrawn

        var groundOffsetX = player.x % 128; // Number of tank tiles on x axis
        var groundOffsetY = player.y % 128; // Number of tank tiles on y axis

        for (var i = -numberOfTiles; i <= numberOfTiles; i++) {
            for (var j = -numberOfTiles; j <= numberOfTiles; j++) {
                groundTiles.addFrame('imgs/imgGround.png', i * tileSizeX, j * tileSizeY);
            }
        }


        // We'll use these later to animate the building
        var animateX = 1;
        var animateY = 0;

        /**
         * We'll draw the building off the players viewport to start to give it the impression we're driving past
         * @type {number}
         */
        var buildingTexture = new PIXI.Texture(
                PIXI.utils.TextureCache['imgs/imgBuildings.png'],
                new PIXI.Rectangle(0, 0, 144, 144, 144)
        );
        groundTiles.addFrame(buildingTexture, (2 * 128), (4 * 128) * -1, animateX, animateY);

        groundTiles.position.set(playerOffsetX + player.x - groundOffsetX, playerOffsetY + player.y - groundOffsetY);
    }

    var tick = new Date().getTime();
    var tileAnim = 0;
    var tileAnimationTick = 0;
    function gameLoop() {


        stats.begin();

        tick = new Date().getTime();
        if (player.y % (10 * tileSizeY ) === -0) {
            console.log("redrawing");
            drawTiles();
        }

        // Make it look like the tank is driving forwards by moving the tiles
        player.y -= 4;
        groundTiles.pivot.set(player.x, player.y);

        app.renderer.plugins.tilemap.tileAnim[0] = tileAnim * 144;
        if (tick > tileAnimationTick) {
            tileAnimationTick = tick + 300;

            tileAnim = tileAnim + 1;
            if (tileAnim >= 3) {

                tileAnim = 0;
            }
        }

        stats.end();
        requestAnimationFrame(gameLoop);
    }

</script>
</body>
</html>
```






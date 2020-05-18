<script>
    import { onMount } from 'svelte';
    let innerWidth = 0;
    let innerHeight = 0;
    let canvasWidth = 500;
    let canvasHeight = 500;
    class Cell {
        constructor() {
            this.state = 0;
        }
    }

    const setDropdownValue = (key, value, openFlag) => {
    }
    const colorSchemes = [
        ["#090910", "#f8f8ff"],
        ["#f9ed69", "#b83b5e"],
        ["#f38181", "#95e1d3"],
        ["#ff2e63", "#252a34"],
        ["#dbe2ef", "#3f72af"],
        ["#fae3d9", "#903749"],
        ["#ebcbae", "#6ef3d6"],
    ];
    const rules = [
        {key: "game_of_life", text: "Game of Life", value: "game_of_life"},
        {key: "n_minus_one", text: "N - 1", value: "n_minus_one"},
    ]
    const objects = [
        {name: "pixel", size: [1,1], activations: [1]},
    ]
    let patterns = [];
    let pixelSize = 1;
    let selectedScheme = colorSchemes[0];
    let selectedRule = rules[0];
    let selectedPattern = null;
    const node = { state: 0, oldState: 0 }
    let grid = [[]]
    const setRandom = () => {
        Array.from({length: 100000}).forEach((value, index) => {
            const row = Math.round(Math.random() * (canvasHeight - 1));
            const column = Math.round(Math.random() * (canvasWidth - 1));
            grid[row][column]["state"] = 1;
            grid[row][column]["oldState"] = 0;
        });

    }
    const initGrid = () => {
        grid = Array.from({length: canvasHeight}, (v, i) => Array.from({length: canvasWidth}, (v, i) => Object.assign({}, node)));
    }
    let startGame = () => {}
    let resetGame = () => {}
    let canvasHover = () => {}
    let clearCanvasHover = () => {}
    let canvasClick = () => {}
    let setColorScheme = () => {}
    let setPixelSize = () => {}
    let dropdownOneOpen = false;
    let dropdownTwoOpen = false;
    const dropdownState = {
        pattern: false,
        colorScheme: false,
    }
	onMount(() => {
        canvasWidth = innerWidth;
        canvasHeight = innerHeight;
		var c = document.getElementById("myCanvas");
		var ctx = c.getContext("2d");
        var ctx = c.getContext("2d");
        const getWrappedCoord = (coord) => {
            const wrappedCoord = [coord[0] % canvasHeight, coord[1] % canvasWidth];
            if (wrappedCoord[0] < 0) {
                wrappedCoord[0] += canvasHeight
            }
            if (wrappedCoord[1] < 0) {
                wrappedCoord[1] += canvasWidth
            }
            return wrappedCoord;
        } 
        const render = () => grid.forEach((row, rowIndex) => {
            var c = 0
            row.forEach((element, elementIndex) => {
                if (element.state === 1 && element.oldState !== 1) {
                    ctx.fillStyle = selectedScheme[0];
                    ctx.fillRect(elementIndex, rowIndex, pixelSize, pixelSize);
                } else if (element.state === 0 && element.oldState !== 0) {
                    ctx.fillStyle = selectedScheme[1];
                    ctx.fillRect(elementIndex, rowIndex, pixelSize, pixelSize);
                }
            })
        });
        setPixelSize = (newSize) => {

            paused = true;
            pixelSize = newSize;
            const x_pixels = Math.round(canvasWidth / pixelSize);
            const y_pixels = Math.round(canvasHeight / pixelSize);
            grid = grid.slice(0,y_pixels);
            grid.forEach(row => {
                row = row.slice(0,x_pixels);
                while (row.length < x_pixels) {
                    row.push(Object.assign({}, node))
                }
            });
            while (grid.length < y_pixels) {
                grid.push(Array.from({length: canvasWidth}, (v, i) => Object.assign({}, node)))
            }
            ctx.clearRect(0,0,canvasWidth,canvasHeight)

            paused = false;
        }
        const neighbors = [[0,1],[0,-1],[1,1],[1,0],[1,-1],[-1,0],[-1,1],[-1,-1]];
        const updateGOL = () => {
            grid.forEach((row, rowIndex) => {
                row.forEach((element, elementIndex) => {
                    const neighborSum = neighbors.reduce((sum, transform) => {
                        const newCoord = getWrappedCoord([rowIndex + transform[0], elementIndex + transform[1]]);
                        sum += grid[newCoord[0]][newCoord[1]].state;
                        return sum;
                    }, 0);
                    element.oldState = element.state;
                    if (neighborSum === 3 || (neighborSum === 2 && element.state === 1)) {
                        element.newState = 1;
                    } else {
                        element.newState = 0;
                    }
                });
            });
            grid.forEach((row, rowIndex) => {
                row.forEach((element, elementIndex) => {
                    element.state = element.newState;
                })
            })
        };
        const updateNMinusOne = () => grid.forEach((row, rowIndex) => {
            row.forEach((element, elementIndex) => {
                element.oldState = element.state;
                const neighbors = [[0,1],[0,-1],[1,1],[1,0],[1,-1],[-1,0],[-1,1],[-1,-1]];
                neighbors.forEach((transform) => {
                    const newCoord = getWrappedCoord([rowIndex + transform[0], elementIndex + transform[1]]);
                    if ((grid[newCoord[0]][newCoord[1]].state === element.state - 1) || (grid[newCoord[0]][newCoord[1]].state === 1 && element.state === 0)) {
                        grid[newCoord[0]][newCoord[1]].state = element.state;
                    }
                });
            })
        });
        const spawnObject = (coord, object) => {
            let counter = 0;
            for (var i = 0; i < object.size[0]; i++) {
                for (var j = 0; j < object.size[1]; j++) {
                    if (object.pattern[i][j] === "O") {
                        grid[coord[0] + i][coord[1]+j].oldState = grid[coord[0] + i][coord[1]+j].state;
                        grid[coord[0] + i][coord[1]+j].state = 1;
                    } else {
                        grid[coord[0] + i][coord[1]+j].oldState = grid[coord[0] + i][coord[1]+j].state;
                        grid[coord[0] + i][coord[1]+j].state = 0;
                    }
                    counter += 1;
                }
            }
        }
        let oldX = null;
        let oldY = null;
        canvasHover = (e) => {
            const rect = ctx.canvas.getBoundingClientRect()
            const newX = e.clientX - rect.left;
            const newY = e.clientY - rect.top;
            ctx.fillStyle = "rgba(99, 183, 11, 0.3)";
            if (oldX) {
            //    ctx.clearRect(oldX, oldY, 11, 11)
            }
            //ctx.fillRect(newX,newY,10,10);
            oldX = newX;
            oldY = newY;
        }
        clearCanvasHover = () => {
            if (oldX) {
            //    ctx.clearRect(oldX, oldY, 11, 11)
                oldX = null;
                oldY = null;
            }
        }
        canvasClick = (e) => {
            const rect = ctx.canvas.getBoundingClientRect()
            const newX = e.clientX - rect.left;
            const newY = e.clientY - rect.top;
            spawnObject([Math.round(newY), Math.round(newX)], selectedPattern)
        }
        setColorScheme = (scheme) => {
            selectedScheme = scheme
            ctx.fillStyle = selectedScheme[1];
            ctx.fillRect(0, 0, canvasWidth, canvasHeight)
        }
        var count = 0;
        let paused = false;
        var lifeLoop = () => {
            count += 1;
            if (count < 10000) {
                if (selectedRule.value === "game_of_life") {
                    updateGOL();
                } else if (selectedRule.value === "n_minus_one") {
                    updateNMinusOne();
                }
                render();
                if (!paused) {
                    requestAnimationFrame(lifeLoop)
                }
            }
        }

        render()
        resetGame = (blank) => {
            count = 0;
            paused = true;
            initGrid();
            if (!blank) {
                setRandom();
            }
            ctx.clearRect(0,0,canvasWidth,canvasHeight);
            render();
            paused = false;
            lifeLoop();
        }
        startGame = () => {
            resetGame()
        }
        setTimeout(startGame, 1000)
        fetch("gol_items.json").then(response => response.json())
           .then(data => {
            patterns = Object.values(data)
            selectedPattern = patterns[0]
        });
	})
</script>

<main>
	<canvas
      id=myCanvas
      width={canvasWidth}
      height={canvasHeight}
      class="gol-canvas"
      style="background-color: {selectedScheme[1]}"
      on:click="{(event) => canvasClick(event)}"
      on:mousemove="{(event) => canvasHover(event)}"
      on:mouseleave="{(event) => clearCanvasHover(event)}"
    />
    <div class="controls">
    <h1>life</h1>
    <p>
        <button class="btn" on:click={() => resetGame(true)}>
            Reset Blank
        </button>
        <button class="btn" on:click={() => resetGame(false)}>
            Reset Random
        </button>
    </p>
    <div class="bottom-controls">
        <div class="dropdowns">
        <div class="dropdown">
            <button class="dropdown-button" on:click="{(event) => dropdownOneOpen = !dropdownOneOpen}">
                Colors
            </button>
            <div class="dropdown-list {dropdownOneOpen ? "open" : "hidden"}">
                {#each colorSchemes as scheme}
                <div class="dropdown-list-item" on:click="{(event) => {
                    setColorScheme(scheme)
                    dropdownOneOpen = false
                }}">
                    <div class="dropdown-option-color" style="background-color: {scheme[0]}"></div>
                    <div class="dropdown-option-color" style="background-color: {scheme[1]}"></div>
                 </div>
                {/each}
            </div>
        </div>
        <div class="dropdown">
            <button class="dropdown-button" on:click="{(event) => dropdownTwoOpen = !dropdownTwoOpen}">
                Spawn Pattern
            </button>
            <div class="dropdown-list {dropdownTwoOpen ? "open" : "hidden"}">
                {#each patterns as pattern}
                <div class="dropdown-list-item pattern-item" on:click="{(event) => {
                    selectedPattern = pattern
                    dropdownTwoOpen = false
                }}">
                    <div class="pattern-name">{pattern.name}</div>
                    
                    <div class="pattern-svg">
                        <svg viewBox="0 0 100 100" width="50" height="50">
                            {@html pattern.svg}
                        </svg>
                    </div>
                 </div>
                {/each}
            </div>
        </div>
        </div>
        <!--input type="range" min=1 max=32 on:change="{(e) => setPixelSize(e.target.value)}" /-->
    </div>
</div>
    
</main>
<svelte:window bind:innerWidth={innerWidth} bind:innerHeight={innerHeight}/>

<style>
    .controls {
        flex-direction: column;
        display: flex;
        position: absolute;
        right: 10px;
        z-index: 1;
    }
    .dropdown {
        position: relative;
        display: inline-block;
        width: 150px;
    }
    .dropdown-button {
        margin: 0;
        position: relative;
        padding: 0;
        width: 100%;
    }
    .dropdown-list {
        position: absolute;
        width: 100%;
        z-index: 10;
        background-color: white;
        max-height: 400px;
        overflow-y: scroll;
    }
    .dropdown-list-item {
        display: flex;
        background-color: "#fefeff";
    }
    .pattern-item {
        flex-direction: column;
        border-bottom: 1px solid lightblue;
    }
    .dropdown-option-color {
        width: 50%;
        height: 16px;
    }
    .dropdown-list-item:hover {
        background-color: lightblue;
        opacity: 50%;
    }
    .dropdown-list.hidden {
        display:none;
    }
    .btn {
        width: 150px;
    }
	main {
		text-align: center;
        font-family: Courier, monospace;
        background-color: "#fefeff";
	}

	h1 {
		color: #aed1d6;
		font-size: 4em;
		font-weight: 120;
        z-index: 1;
        margin: 0;
        padding: 0;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

    colorscheme-color {
        width: 6px;
        height: 6px;
        display: float;
    }
    canvas {
        position: absolute;
        top: 0;
        left: 0;
        margin: 0;
        padding: 0;
    }
</style>
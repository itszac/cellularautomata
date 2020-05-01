<script>
    import { onMount } from 'svelte';
    class Cell {
        constructor() {
            this.state = 0;
        }
    }
    const node = {state: 0, oldState: 0}
	const baseRow = Array.from({length: 1000}, (v, i) => Object.assign({}, node));
    const grid = Array.from({length: 1000}, (v, i) => Array.from({length: 1000}, (v, i) => Object.assign({}, node)))
    console.log(grid)
    Array.from({length: 100000}).forEach((value, index) => {
        const row = Math.round(Math.random() * 999);
        const column = Math.round(Math.random() * 999);
        grid[row][column]["state"] = 1;
    });
	onMount(() => {
		var c = document.getElementById("myCanvas");
		var ctx = c.getContext("2d");
        var count = 0
        console.log(count)
        var ctx = c.getContext("2d");
        const getWrappedCoord = (coord) => {
            const wrappedCoord = [coord[0] % 1000, coord[1] % 1000];
            if (wrappedCoord[0] < 0) {
                wrappedCoord[0] += 1000
            }
            if (wrappedCoord[1] < 0) {
                wrappedCoord[1] += 1000
            }
            return wrappedCoord;
        } 
        const render = () => grid.forEach((row, rowIndex) => {
            var c = 0
            row.forEach((element, elementIndex) => {
                if (element.state === 1 && element.oldState !== 1) {
                    ctx.fillRect(rowIndex, elementIndex, 1, 1);
                } else if (element.state === 0 && element.oldState !== 0) {
                    ctx.clearRect(rowIndex, elementIndex, 1, 1);
                }
            })
        });
        const updateGOL = () => grid.forEach((row, rowIndex) => {
            row.forEach((element, elementIndex) => {
                const neighbors = [[0,1],[0,-1],[1,1],[1,0],[1,-1],[-1,0],[-1,1],[-1,-1]];
                const neighborSum = neighbors.reduce((sum, transform) => {
                    const newCoord = getWrappedCoord([rowIndex + transform[0], elementIndex + transform[1]]);
                    sum += grid[newCoord[0]][newCoord[1]].state;
                    return sum;
                }, 0);
                element.oldState = element.state;
                if (neighborSum === 3 || neighborSum === 2 && element.state === 1) {
                    element.state = 1;
                } else {
                    element.state = 0;
                }
            });
        });
        var x = () => {
            count += 1;
            if (count < 10000) {
                updateGOL();
                render();
                requestAnimationFrame(x)
            }
        }
        render()
        console.log('rendered')
        setTimeout(x, 5000)
	})
</script>

<main>
	<h1>cellular automata</h1>
	<canvas id=myCanvas width=1000 height=1000 />
    <p>
        todo:
          pixel object? 
          get game of life running on canvas
          add either spawning of items from game of life zoo or ability to click to init
          add sliders for canvas size
    </p>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
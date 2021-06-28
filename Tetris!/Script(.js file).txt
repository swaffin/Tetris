document.addEventListener('DOMContentLoaded', () => {
  //Sets up the size of the chart.
  const GRID_WIDTH = 10
  const GRID_HEIGHT = 20
  const GRID_SIZE = GRID_WIDTH * GRID_HEIGHT

  //Creates all of the div tags from the html file.
  const grid = createGrid();
  let squares = Array.from(grid.querySelectorAll('div'))
  const startBtn = document.querySelector('.button')
  const hamburgerBtn = document.querySelector('.toggler')
  const menu = document.querySelector('.menu')
  const span = document.getElementsByClassName('close')[0]
  const scoreDisplay = document.querySelector('.score-display')
  const linesDisplay = document.querySelector('.lines-score')
  let currentIndex = 0
  let currentRotation = 0
  const width = 10
  let score = 0
  let lines = 0
  let timerId
  let nextRandom = 0
  const colors = [
    'url(images/blue_block.png)',
    'url(images/pink_block.png)',
    'url(images/purple_block.png)',
    'url(images/peach_block.png)',
    'url(images/yellow_block.png)'
  ]

  /*This creates our grid. I did this by using the div tags (in our html file) and then made an array in javascript for them.
  I did this using the "document.createElement" method.
  */
  function createGrid() {
    // the main grid
    let grid = document.querySelector(".grid")
    for (let i = 0; i < GRID_SIZE; i++) {
      let gridElement = document.createElement("div")
      grid.appendChild(gridElement)
    }
    //This places the shapes on the grid
    for (let i = 0; i < GRID_WIDTH; i++) {
      let gridElement = document.createElement("div")
      gridElement.setAttribute("class", "block3")
      grid.appendChild(gridElement)
    }
    let previousGrid = document.querySelector(".previous-grid")
    for (let i = 0; i < 16; i++) {
      let gridElement = document.createElement("div")
      previousGrid.appendChild(gridElement);
    }
    return grid;
  }

  //assign functions to keycodes
  function control(e) {
    if (e.keyCode === 39)
      moveright()
    else if (e.keyCode === 38)
      rotate()
    else if (e.keyCode === 37)
      moveleft()
    else if (e.keyCode === 40)
      moveDown()
  }

  // the classical behavior is to speed up the block if down button is kept pressed so doing that
  document.addEventListener('keydown', control)
  //Creates the movement with the shapes.
  //L shape
  const Lblock = [
    [1, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1, 2],
    [GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH * 2 + 2],
    [1, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1, GRID_WIDTH * 2],
    [GRID_WIDTH, GRID_WIDTH * 2, GRID_WIDTH * 2 + 1, GRID_WIDTH * 2 + 2]
  ]
  //Z shape
  const Zblock = [
    [0, GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1],
    [GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH * 2, GRID_WIDTH * 2 + 1],
    [0, GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1],
    [GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH * 2, GRID_WIDTH * 2 + 1]
  ]
  //T shape
  const Tblock = [
    [1, GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH + 2],
    [1, GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH * 2 + 1],
    [GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH * 2 + 1],
    [1, GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1]
  ]
  //O shape
  const Oblock = [
    [0, 1, GRID_WIDTH, GRID_WIDTH + 1],
    [0, 1, GRID_WIDTH, GRID_WIDTH + 1],
    [0, 1, GRID_WIDTH, GRID_WIDTH + 1],
    [0, 1, GRID_WIDTH, GRID_WIDTH + 1]
  ]
  //I shape
  const Iblock = [
    [1, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1, GRID_WIDTH * 3 + 1],
    [GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH + 3],
    [1, GRID_WIDTH + 1, GRID_WIDTH * 2 + 1, GRID_WIDTH * 3 + 1],
    [GRID_WIDTH, GRID_WIDTH + 1, GRID_WIDTH + 2, GRID_WIDTH + 3]
  ]
  //Creates an array with all of the shapes inside of it.
  const Shapes = [Lblock, Zblock, Tblock, Oblock, Iblock]

  //Selects a random shape to place on grid.
  let random = Math.floor(Math.random() * Shapes.length)
  let current = Shapes[random][currentRotation]


  //Makes the shapes move down
  let currentPosition = 4
  //Places/Draws shape on the grid.
  function draw() {
    current.forEach(index => {
      squares[currentPosition + index].classList.add('block')
      squares[currentPosition + index].style.backgroundImage = colors[random]
    })
  }
  function undraw() {
    current.forEach(index => {
      squares[currentPosition + index].classList.remove('block')
      squares[currentPosition + index].style.backgroundImage = 'none'
    })
  }

  //Creates a loop for the shapes to move down via a timer
  function moveDown() {
    undraw()
    currentPosition = currentPosition += width
    draw()
    freeze()
  }
  startBtn.addEventListener('click', () => {
    if (timerId) {
      clearInterval(timerId)
      timerId = null
    } else {
      draw()
      timerId = setInterval(moveDown, 1000)
      nextRandom = Math.floor(Math.random() * Shapes.length)
      displayShape()
    }
  })
  //Let's the shape move to the right, without it shape cannot not move to the right.
  function moveright() {
    undraw()
    const isAtRightEdge = current.some(index => (currentPosition + index) % width === width - 1)
    if (!isAtRightEdge) currentPosition += 1
    if (current.some(index => squares[currentPosition + index].classList.contains('block2'))) {
      currentPosition -= 1
    }
    draw()
  }

  //Let's the shape move to the left, without it shape cannot not move to the left.
  function moveleft() {
    undraw()
    const isAtLeftEdge = current.some(index => (currentPosition + index) % width === 0)
    if (!isAtLeftEdge) currentPosition -= 1
    if (current.some(index => squares[currentPosition + index].classList.contains('block2'))) {
      currentPosition += 1
    }
    draw()
  }

  //Creates a function for the shape to stay put if it hits an object.
  function freeze() {
    // if block has settled
    if (current.some(index => squares[currentPosition + index + width].classList.contains('block3') || squares[currentPosition + index + width].classList.contains('block2'))) {
      // make it block2
      current.forEach(index => squares[index + currentPosition].classList.add('block2'))
      // start a new shape to make it start falling
      random = nextRandom
      nextRandom = Math.floor(Math.random() * Shapes.length)
      current = Shapes[random][currentRotation]
      currentPosition = 4
      draw()
      displayShape()
      addScore()
      gameOver()
    }
  }
  freeze()

  //Allows the user to rotate the shape, without it you cannot rotate the shape.
  function rotate() {
    undraw()
    currentRotation++
    if (currentRotation === current.length) {
      currentRotation = 0
    }
    current = Shapes[random][currentRotation]
    draw()
  }

  //If you lose you will get a game over or end which stops the game.
  function gameOver() {
    if (current.some(index => squares[currentPosition + index].classList.contains('block2'))) {
      scoreDisplay.innerHTML = 'end'
      clearInterval(timerId)
    }
  }

  //show previous shapes in scoreDisplay
  const displayWidth = 4
  const displaySquares = document.querySelectorAll('.previous-grid div')
  let displayIndex = 0
  const smallBlocks = [
    [1, displayWidth + 1, displayWidth * 2 + 1, 2], /* lshape! */
    [0, displayWidth, displayWidth + 1, displayWidth * 2 + 1], /* Zshape! */
    [1, displayWidth, displayWidth + 1, displayWidth + 2], /* Tshape! */
    [0, 1, displayWidth, displayWidth + 1], /* Oshape! */
    [1, displayWidth + 1, displayWidth * 2 + 1, displayWidth * 3 + 1] /* Ishape!*/
  ]
  function displayShape() {
    displaySquares.forEach(square => {
      square.classList.remove('block')
      square.style.backgroundImage = 'none'
    })
    smallBlocks[nextRandom].forEach(index => {
      displaySquares[displayIndex + index].classList.add('block')
      displaySquares[displayIndex + index].style.backgroundImage = colors[nextRandom]
    })
  }

  //Creates a scoreboard to be tracked.
  function addScore() {
    for (currentIndex = 0; currentIndex < GRID_SIZE; currentIndex += GRID_WIDTH) {
      const row = [currentIndex, currentIndex + 1, currentIndex + 2, currentIndex + 3, currentIndex + 4, currentIndex + 5, currentIndex + 6, currentIndex + 7, currentIndex + 8, currentIndex + 9]
      if (row.every(index => squares[index].classList.contains('block2'))) {
        score += 10
        lines += 1
        scoreDisplay.innerHTML = score
        linesDisplay.innerHTML = lines
        row.forEach(index => {
          squares[index].style.backgroundImage = 'none'
          squares[index].classList.remove('block2') || squares[index].classList.remove('block')
        })
        const squaresRemoved = squares.splice(currentIndex, width)
        squares = squaresRemoved.concat(squares)
        squares.forEach(cell => grid.appendChild(cell))
      }
    }
  }
  hamburgerBtn.addEventListener('click', () => {
    menu.style.display = 'flex'
  })
  span.addEventListener('click', () => {
    menu.style.display = 'none'
  })

})






























































/*
If you're reading this that means you're Miss.Schapelhouman! I just wanted to say thank for the awesome school year!!!
You're one the best teachers I've ever had and I wanted to let you know you're going to be an awesome teacher, I know that's something you already know but still thought you should know that (:
Oh! Also, I got paid for the bash script (with the virus written in python)! So exciting, it is my first time being paid for code!!!!!!!! I would send it to it if you wanted to look at it but I don't know if I actually own it anymore, like I made it but he baught it so is that his? idk doesn't really matter!! I got money for my code, MY CODE!! idk very exciting for me hehe 😎
*/
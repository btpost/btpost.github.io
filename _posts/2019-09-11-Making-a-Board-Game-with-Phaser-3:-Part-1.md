---
title: Making a Board Game with Phaser 3 - Part 1
topic: Programming
---
Today, I finished a playable demo of the board game Quoridor in JavaScript using the game development framework Phaser 3. Here's the **[Wikipedia page for Quoridor](https://en.wikipedia.org/wiki/Quoridor)** if you've never heard of it. So far, the game can be played with two players locally sharing a mouse. I'm hoping to add more features to the game in the future, which is why this post is part 1.

The purpose of this post is to explain what I've done, and point out some places I could improve my code so far. I think it'd be useful for someone else trying to build a boardgame with Phaser 3, since I couldn't find any examples. What I've done is a bit of a hacky way to do it, but it works well. You can find the full project on my GitHub -- here's a **[link](https://github.com/btpost/Quoridor)**. To run it yourself, you're going to have to set up a server and use it in browser. I just used Python's SimpleHTTPServer, but there are a wide variety of options. See the Phaser 3 Docs for a **[set up guide](https://phaser.io/tutorials/getting-started-phaser3)** if you're unsure.

Now, down to business. I've got my code open, and I'm just going to walk my way down index.html and explain the relevant parts as they come up. First, we've got the head tag.

{% highlight html %}
<!doctype html> 
<html lang="en"> 
<head> 
    <meta charset="UTF-8" />
    <title>Quoridor</title>
    <script src="//cdn.jsdelivr.net/npm/phaser@3.11.0/dist/phaser.js"></script>
    <script type="module" src="js/square.js"></script>
    <script type="module" src="js/board.js"></script>
    <script type="module" src="js/pawn.js"></script>
    <script type="module" src="js/player.js"></script>
    <style type="text/css">
        body {
            margin: 0;
        }
    </style>
</head>
{% endhighlight %}

Here I'm getting the Phaser 3 library with the first script tag. If you want to make a Phaser 3 game, you'll need that tag in the head. The other 4 script tags are "classes" (technically normal JavaScript objects, classes are just syntactic sugar) for use in the project. In a (semi-successful) attempt to improve readability, I put them into their own separate files and imported them. More on that later. Finally, you can see me messing with the css. Hopefully, this is straight forward.

Next, we have some imports, corresponding to the script tags above. Then, we get to the config.

{% highlight javascript %}
var config = {
    width: 630,
    height: 630,
    type: Phaser.AUTO,
    parent: 'phaser-example',
    scene: {
    	preload: preload,
        create: create,
        update: update
    }
};
{% endhighlight %}

Phaser 3 uses the config to decide how to set up the game window and render the visual/audio components. *Width* and *height* are the size of the game in the browser. *Type* allows you to change how the video and audio is rendered. *Parent* allows you to inherit another config, so I inherited the config for the phaser examples posted in their docs. *Scene* allows you to set a preload function, a create function, and an update function. Since this project only uses one scene so far, I've named the functions preload, create, and update, but I could set the fields in the scene object to any function I write in the code below. I'll go over each function as I get to them.

Next, I defined some globals to use throughout the program. These are all objects that are used in more than one of the functions below, and are used to keep track of the state of the game. I'll explain them as they come up.

{% highlight javascript %}
var game = new Phaser.Game(config);
var wall;
var pawn1;
var pawn2;
var turn = 0;
var wall_mode = 0;
var player_num = 2;
{% endhighlight %}

Moving on to the preload function. The preload function is mostly used to load assets into your game. I load in three images. It's a pretty simple game.

{% highlight javascript %}
function preload()
{
	this.load.image('square', 'assets/board_square.png');
	this.load.image('pawn', 'assets/pawn.png');
	this.load.image('wall', 'assets/wall.png');
}
{% endhighlight %}

The create function is used to initialize the games objects and layout the general logical flow of the game. This is the bulk of the code in the index, so I'm going to go through it in chunks.

{% highlight javascript %}
function create ()
{
    // Initialize mouse input, and turn of right click menu
  	var pointer = this.input.activePointer;
  	this.input.mouse.disableContextMenu();
    
    // Initialize the game board
   	var board = new Board({scene: this});
   	console.log(board);
    
    // Initialize pawns. Turn player 2's pawn red
    var pawn1 = new Pawn({scene: this, x: 315, y: 35, board: board});
    var pawn2 = new Pawn({scene: this, x: 315, y:595, board: board});
    pawn2.tint = 0xaa0000;

    // Initialize players
	var player1 = new Player({board: board, pawn: pawn1, wall_count: 10, cpu: false});
	var player2 = new Player({board: board, pawn: pawn2, wall_count: 10, cpu: false});
    var players = [player1, player2]

    // Create a local variable for the current scene to pass into the event handler below
    var _this = this;
{% endhighlight %}

As you can see by the comments, this is mostly initializing the different objects for use in the game. After I set up the mouse input, you can see instances of three of the classes that I made for this project. I'll start by explaining the class that isn't shown here, *Square*, since it's an important part of the *Board* and *Pawn* class. 

{% highlight javascript %}
export class Square extends Phaser.GameObjects.Image {

	constructor(config) {
		super(config.scene, config.x, config.y, 'square');
		// Added fields for the array coordinates of the square
		this.arr_x = Math.floor(config.x/70);
		this.arr_y = Math.floor(config.y/70);
		// Add array for keeping track of which sides of the square are blocked
		this.blocked_sides = new Array(4).fill(0);
		this.has_pawn = false;


		// Add the square to the screen
		config.scene.add.existing(this);


	}
{% endhighlight %}

The square class is an extension of Phaser 3's image class, and it is what I used to represent each square on the board. By looking at the constructor, you can see that I've added a few extra instance variables to keep track of where each square is in the *Board* array. Each square is 70x70 px, so in order to give it the correct array position, I divide it's on screen position by 70 and then round down (Phaser measures position from the center, so the floor call is necessary). I set up an array to keep track of what sides of the square are blocked by walls that have been played, and I gave each square a boolean to keep track of whether a pawn is on it or not. The last line adds the square into the scene. 

{% highlight javascript %}
	// Methods for blocking each side of the square
	block_up() {
		this.blocked_sides[0] = 1;
	}
	block_down() {
		this.blocked_sides[1] = 1;
	}
	block_left() {
		this.blocked_sides[2] = 1;
	}
	block_right() {
		this.blocked_sides[3] = 1;
	}

	get block_array (){
		return this.blocked_sides;
	}

	blocked (i) {
		if(this.blocked_sides[i] == 1)
		{
			return true;
		}
		return false;
	}

}

export default Square;
{% endhighlight %}

The methods for square are pretty straightforward. They're basically just getters and setters, outside of *blocked*, which will check a particular index in the blocked_sides array to see if there's a wall. A big thing I think I could improve here is giving each square it's own event listener for being clicked. Right now, the event handler for clicking on screen finds the square that was clicked by using the position of the mouse, which can lead to a disconnect between what's on screen and what's being done in the code. It'd also make the code a bit more readable by getting rid of all the Math.floor(/70)'s.

Now, that we're acquainted with *Square*, we can take a look at *Pawn*.

{% highlight javascript %}
export class Pawn extends Phaser.GameObjects.Sprite {
	constructor(config) {
		super(config.scene, config.x, config.y, 'pawn');
		this.arr_x = Math.floor(config.x/70);
		this.arr_y = Math.floor(config.y/70);
		this.square = config.board.get_square(this.arr_x, this.arr_y);
		this.square.has_pawn = true;
		this.board = config.board;

		config.scene.add.existing(this);
	}
{% endhighlight %}

Pawn extends the sprite object, because I have a feeling I might need some of the sprite functionality in the future, but for now it's not doing anything an image couldn't do. Again, it has instance variables for its array position, as well as a square that it's on, and a board it's on. The rest of the class is a single method: *move*. The comments here do a pretty good job explaining what's going on.
{% highlight javascript %}
	// Check to see if a valid space was clicked. If so, move the pawn to the space.
	move (x, y) {
		// Convert the input coords to array coords
		var arr_x = Math.floor((x)/70);
		var arr_y = Math.floor((y)/70);
		var new_square = this.board.get_square(arr_x, arr_y);

		// Check to see if the square clicked is 1 space up, 
		// down, left, or right, and if there's a pawn there.
		if(Math.abs(arr_x - this.square.arr_x) <= 1 && Math.abs(arr_y - this.square.arr_y) <= 1 
		&& Math.abs(arr_x - this.square.arr_x)+Math.abs(arr_y - this.square.arr_y) <= 1 
		&& !(new_square.has_pawn))
		{
			// Check to see if the current square is block in the direction of the new square
			// if it isn't blocked, set the new square to this pawn's current square

			// Checking for moving up
			if(arr_y < this.square.arr_y)
			{
				if(!(this.square.blocked(0)))
				{	
					this.square.has_pawn = false;
					this.square = new_square;
					this.arr_x = new_square.arr_x;
					this.arr_y = new_square.arr_y;
					this.x = new_square.x;
					this.y = new_square.y;
					this.square.has_pawn = true;
					return true;
				}
			}
			// Checking for moving down
			else if (arr_y > this.square.arr_y)
			{
				if(!(this.square.blocked(1)))
				{
					this.square.has_pawn = false;
					this.square = new_square;
					this.square.has_pawn = true;
					this.arr_x = new_square.arr_x;
					this.arr_y = new_square.arr_y;
					this.x = new_square.x;
					this.y = new_square.y;
					return true;
				}
			}
			// Checking for moving left
			else if (arr_x < this.square.arr_x)
			{
				if(!(this.square.blocked(2)))
				{
					this.square.has_pawn = false;
					this.square = new_square;
					this.square.has_pawn = true;
					this.arr_x = new_square.arr_x;
					this.arr_y = new_square.arr_y;
					this.x = new_square.x;
					this.y = new_square.y;
					return true;
				}
			}
			// Checking for moving right
			else if (arr_x > this.square.arr_x)
			{
				if(!(this.square.blocked(3)))
				{
					this.square.has_pawn = false;
					this.square = new_square;
					this.square.has_pawn = true;
					this.arr_x = new_square.arr_x;
					this.arr_y = new_square.arr_y;
					this.x = new_square.x;
					this.y = new_square.y;
					return true;
				}
			}	
		}
	}
}

export default Pawn;
{% endhighlight %}

At the bottom, I'm setting a default export, because JavaScript doesn't like it when you have a module without a default export.

On to *Board*. 

{% highlight javascript %}
export class Board {
	constructor(config) {
		this.squares = []
		for (var i = 0; i < 9; i++)
		{
			var row = [];
			for(var j = 0; j < 9; j++)
			{
				row[j] = new Square({scene: config.scene, x: (j*70+35), y: (i*70+35)});
			}
			this.squares[i] = row;
		}
	}
	get_square(x, y) {
		return this.squares[y][x];
	}
{% endhighlight %}

*Board* starts by creating a 9x9 array of squares. This is it's only data. The get_square method is notable only because I have x and y flipped to make addressing the array more intuitive. In a 2d array, the first index tells you which row you're on, which is equivalent to the y-value, and opposite the order of (x,y) coordinates. It makes things easier for me to read, at least.

There are two big methods in the *Board* class, *check_wall_placement* and *add_wall*. I'll go over each of them, starting with *check_wall_placement*. It takes in the (x,y) coordinates from a mouse click on the game board, and the *wall_mode* global variable. *Wall_mode* keeps track of whether a wall is being placed or not, and what orientation the wall should be. The first if statement is checking whether the location clicked is at the edge of the board, since it would be pointless to place a wall along the edges. Since walls are placed between the squares, each wall location has four squares adjacent to it. I make a variable for each of the arrays corresponding to the sides of these squares. 

{% highlight javascript %}
	// Takes in x,y coords from clicking the screen
	// returns true if wall can be placed, false if it cannot be placed
	check_wall_placement(x, y, wall_mode) {
		// For each square, the top is in index 0, bottom 1, left 2, right 3

		x = Math.round(x/70)-1;
		y = Math.round(y/70)-1;

		if(x < 0 || y < 0 || x > 8 || y > 8)
		{
			return false;
		}

		var t_left = this.squares[y][x].block_array;
		var t_right = this.squares[y][x+1].block_array;
		var b_left = this.squares[y+1][x].block_array;
		var b_right = this.squares[y+1][x+1].block_array;

		if(wall_mode === 1)
		{
			if(t_left[3] == 1 || t_right[2] == 1 || b_left[3] == 1 || b_right[2] == 1)
			{
				return false;
			}

			if((t_left[1] == 1 && t_right[1] == 1) || (b_left[0] == 1 && b_right[0] == 1))
			{
				return false
			}

			return true;
		}
		else if (wall_mode === 2)
		{
			
			if(t_left[1] == 1 || t_right[1] == 1 || b_left[0] == 1 || b_right[0] == 1)
			{
				return false
			}

			if((t_left[3] == 1 && b_left[3] == 1) || (t_right[2] == 1 && b_right[2]== 1 ))
			{
				return false;
			}

			return true;
		}
	}
{% endhighlight %}

Next, I take a look at the *wall_mode* to see if the wall is being placed vertically (represented by 1) or horizontally (represented by 2). The first check for each if-statement looks to see if there's a wall going the same direction at the location clicked. For example, if I was trying to place a vertical wall where there is already a vertical wall, then right side of top or bottom left squares would be blocked. The second check verifies there isn't a perpendicular wall at the location. This code could be made more readable by making getters for each of the sides of a square, instead of accessing the blocked_sides array directly.

The *add_wall* method is what actually changes the values of the block array for each of the adjacent squares. The logic I outlined above for determining a wall's location in relation to the squares on the board get's used again here, in a slightly more readable fashion thanks to the *Square* classes setter methods.

{% highlight javascript %}
	// Changes the blocking values on the squares on the board
	// Takes in x,y coords from clicking the screen
	add_wall(x, y, wall_mode) {
		x = Math.round(x/70)-1;
		y = Math.round(y/70)-1;

		var t_left = this.squares[y][x];
		var t_right = this.squares[y][x+1];
		var b_left = this.squares[y+1][x];
		var b_right = this.squares[y+1][x+1];

		if(wall_mode === 1)
		{
			t_left.block_right();
			b_left.block_right();
			t_right.block_left();
			b_right.block_left();
		}
		else if (wall_mode === 2)
		{
			t_left.block_down();
			t_right.block_down();
			b_left.block_up();
			b_right.block_up();
		}
	} 
{% endhighlight %}

Now that we've covered the classes that determine what's drawn on the screen, we can touch on the *Player* class. For now, all this does is keep track of some relevant data, but moving forward in the project, it will (probably) hold the code for AI players, among other things. The *wall_count* variable keeps track of how many walls each player can play, and it is decremented when walls are played. It'd probably be better to give it some mutator methods instead of accessing this data in index.html.

{% highlight javascript %}
export class Player {
	constructor(config) {
		this.board = config.board;
		this.pawn = config.pawn;
		this.wall_count = config.wall_count;
		this.cpu = config.cpu;
	}
{% endhighlight %}

Alright, now back to index.html. We left off in the create method, right before my event handler. One of the biggest problems with this project right now is that there's only one event handler. The code would be much more readable if there were more event handlers, but I wanted to get a working prototype together as quickly as I could, so here we are. It's a work in progress. Anyway, first I'm checking to see if the right mouse button was clicked. If that's the case, then it changes the *wall_mode* from 0 (move pawn) to 1 (vertical wall) to 2 (horizontal wall). If we're moving a pawn, then we access the array of players at the index of the player whose turn it is and move their pawn. The players array, player_num and turn are all globals used to figure out whose turn it is with modulus. The move method returns true if the pawn could be moved, which means the player's turn is over and we move on to the next turn.

{% highlight javascript %}
// Create an event handler for mouse input
    this.input.on('pointerdown', function (pointer) {
        // Change wall display state on right click
        if (pointer.rightButtonDown()){
        	if(wall_mode === 0)
        	{
        		wall_mode = 1;
        	} else if (wall_mode === 1) {
        		wall_mode = 2;
        	} else if (wall_mode === 2) {
        		wall_mode = 0;
        	} else {
        		console.log('Theres a problem with wall_mode');
        	}
        } // Move pawn if not in wall mode
        else if (wall_mode === 0)
        {   
        	if(players[turn%player_num].pawn.move(pointer.worldX, pointer.worldY))
        	{
        		turn++;
        	} 
        } 
        else
        {
        	// Check to see if a wall can be placed at the clicked on spot
        	// Make sure the player clicking has walls to play
        	if(board.check_wall_placement(pointer.worldX, pointer.worldY, wall_mode)
        	 && players[turn%player_num].wall_count != 0)
        	{
        		// Add the wall to the board
        		board.add_wall(pointer.worldX, pointer.worldY, wall_mode);
        		// Draw the wall on the screen
        		var new_x = (Math.round(pointer.worldX/70)*70);
        		var new_y = (Math.round(pointer.worldY/70)*70);
        		var new_wall = _this.add.sprite(new_x, new_y, 'wall');
        		// If it was placed horizontally, draw it sideways
        		if (wall_mode === 2)
        		{
        			new_wall.angle = 90;
        		}
        		players[turn%player_num].wall_count--;
        		turn++; 
        	}
        }
    });
    wall = this.add.sprite(0, 0, 'wall');
}
{% endhighlight %}

In the case that we're placing a wall, we use *Board*'s wall placing methods. Finally, the last thing we do in the create method is make a sprite for the wall. And that's the create method.

The last method is update. We've pretty well set up the games logic in create, so all update does is decide whether we should draw a wall on the screen, and if so, should it be vertical or horizontal. 

{% highlight javascript %}

function update ()
{
	// Initialize mouse input
    var pointer = this.input.activePointer;
	// Change the display of the wall piece based on the wall_mode value
	if (wall_mode == 0)
	{
		wall.setActive(false).setVisible(false);
	}
	else if(wall_mode == 1)
	{
		wall.setActive(true).setVisible(true);
		wall.angle = 0;
		wall.setX(pointer.worldX);
		wall.setY(pointer.worldY);
	} 
	else if (wall_mode == 2)
	{
		wall.setActive(true).setVisible(true);
		wall.angle = 90;
		wall.setX(pointer.worldX);
		wall.setY(pointer.worldY);
	}
}

{% endhighlight %}

And that's all the code in the project. This was a good exercise, and I have a much clearer idea of where to go from here with this project. The next few steps are adding winning and losing states, adding AI, and hopefully setting up online play. If you're reading this, hopefully you found it of some use.
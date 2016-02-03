[![Downloads][downloads-image]][downloads-url] [![Auto Test Status][travis-image]][travis-url] [![license][license-image]][license-url] [![Gitter chat][gitter-image]][gitter-url]

Minimax implementation using AlphaBeta in Node using *asynchronous* calls to customizable game logic, scoring, and move generation.  Because of its *asynchronous* nature, AlphaBeta is *non-blocking*.

The rational and motivation to use asynchronous calls (specifically to the scoring function) is to support integration with other processes such as DBs and REST calls whereby a scoring function uses a growing data lookup.  This cannot be accomplished in a synchronous way in javascript.

Section Links : [Construction](#construction) , [Execution](#execution) , [Examples](#example) , [FAQ](#faq) , [Related](#related-ai-projects) , and [References](#references)

# Construction

### AlphaBeta constructor
```js
var AlphaBetaConstructor = require('alphabeta')
var alphabeta = AlphaBetaConstructor( config )
```
Here is the full configuration object and the constructor in-lined.
```js
var config = {
	scoreFunction 		: scoreFunction,
	generateMoves		: generateMovesFunction,
	checkWinConditions 	: checkWinConditionsFunction,
	uniqueKey			: uniqueKeyFunction,
	state 				: yourInitialStateObject,
	depth 				: theDepthOfSearch
}
var AlphaBetaConstructor = require('alphabeta')
var alphabeta = AlphaBetaConstructor( config )
```

That creates one instance of an AlphaBeta calculator which uses the initial configuration you supply.  All configuration options are optional.  If you want to make two different computer opponents battle eachother using two different strategies you'll want to create two instances of AlphaBeta each with its own configuration.

### alphabeta.setup( config )
> Returns alphabeta

Each new turn or new problem will require you to set the current state of AlphaBeta.  Instead of creating a whole new alphabeta you can reuse the old instance and change any of the configuration parameters you choose.  Here is an example of just chaning the state configuration parameter.

```js
config = {
	state : anotherStateObject
}

alphabeta.setup( config )
```
Or if you like one-liners
```js
alphabeta.setup( { state : anotherStateObject } )
```
### alphabeta.clone( config )
> Returns a new alphabeta

If you want another alphabeta based on the configuration of an existing alphabeta but with slight changes use the .clone method and pass in only the configuration changes you want.  This is different than *.setup* in that it creates a whole new alphabeta and does not change the current alphabeta.  In this example the search depth is changed to 7:

```js
config = {
	depth : 7
}

var anotherAlphaBeta = alphabeta.clone( config )
```
Again, you can one-liner this
```js
var anotherAlphaBeta = alphabeta.clone( { depth : 7 } )
```

# Execution

### alphabeta.step( callback )
> Returns alphabeta

Call the AlphaBeta calculator like so:
```js
alphabeta.step( function( beststate ) {
	if ( beststate === undefined ) {
		// do something like call alphabeta.step again
	} else {
		// beststate is the best next state as generated by generateMoves
		// or null if a next state cannot be calculated
	}
})
```

'step' moves the calculator ahead by one step.  Depending on the number of moves generated and the depthParameter there could be hundreds, thousands, millions, or more steps needed before the calculator finishes.  alphabeta.best() returns the best state.

### alphabeta.allSteps( callback )
> Returns alphabeta

To execute all the steps until AlphaBeta has found the best move for the depth.  *.allSteps* is *non-blocking* which means your UI will keep working.  That's a good thing.  Call like so:
```js
alphabeta.allSteps( function( beststate ) {
	// beststate is the best next state as generated by generateMoves
	// or null if a next state cannot be calculated
})
```

### alphabeta.stepForMilliseconds( milliseconds , callback )
> Returns alphabeta

To execute all the steps until AlphaBeta has found the best move for the depth or the number of milliseconds has expired.  Like *.allSteps*, this is also *non-blocking*.  Call like so:

```js
alphabeta.stepForMilliseconds( milliseconds , function( beststate ) {
	if ( beststate != undefined ) {
		// beststate is the best next state as generated by generateMoves
		// or null if a next state cannot be calculated
	} else {
		// a best next state has not yet been found.
		// You can continue the search using 
		// step, allSteps, or stepForMilliseconds.
	}
})
```

### alphabeta.incrimentDepthForMilliseconds( milliseconds , callback )
> Returns alphabeta

Execute all the steps of AlphaBeta like *.stepForMilliseconds*.  If time premits iteratively try larger depths.  This function is especially useful for a *just in time* methodology such that you only have a given time to get the best aswer you can and want to reach the highest depth possible in that time.  Note : the callback does not return a 'beststate'.  Like *.allSteps*, this is also *non-blocking*.  Call like so:

```js
alphabeta.incrimentDepthForMilliseconds( milliseconds , function( result ) {

	// result.alphabeta is a clone of alphabeta 
	//    which successfully executed *.stepForMilliseconds*
	// result.depth is the depth of the clone



	if ( result.alphabeta && result.alphabeta.best() != undefined ) {
		var beststate = result.alphabeta.best()
	} else {
		// not enough time to find any result.
	}
})
```
The full result object is as follows
```js
var result = {
	alphabeta :	// the AlphaBeta object that successfully completed
	depth :		// the search depth

	incomplete : {	// only exists if depth > 1

		alphabeta :	// the AlphaBeta that was incomplete.
					// Use *.step* functions on this to
					// continue the search were it was left off.
		depth :		// the search depth of the incomplete alphabeta
	}
}
```
### Execution results

If you want to know the best score found you can use 

```js
var score = alphabeta.alpha()
```

If you want to know the predicted final state if the everything unfolds as the AlphaBeta calculator predicts use:

```js
var state = alphabeta.prediction()
```


# Configuration
This is the specification of the configuration functions you pass to AlphaBeta

### scoreFunction( state , callback )
> Return value ignored

The scoreFunction that you provide is an asynchronous function that evaluates a state like so:

```js
function yourScoreFunction( state , scoreCallback ) {
	var scoreNumber = 0
	// inspect state and modify the score
	scoreCallback( scoreNumber )
}
```

### generateMoves( state )
> Must return a list of states or the empty list '[]' if no moves are available

The generateMovesFunction that you provide is a synchronous function that returns a list of possible states like so:

```js
function yourGenerateMovesFunction( currentState ) {
	var nextPossibleStates = []

	// use the currentState and possibly some 
	// other info to create new state objects
	// which would represent valid next states.
	// If this is a game, then the state
	// is the game board and move as an object

	// for ( item in a list of possible moves ) {
		// use item to create a new state
		// push state onto nextPossibleStates
	// }

	return nextPossibleStates

}
nextPossibleStates = yourGenerateMovesFunction( currentState )
```

### checkWinConditions( state )
> Must return falsy if not a win condition, truthy otherwise

The checkWinConditions function that you provide is a synchronous function that checks to see if the state is a good end state such as a winning move.  A psudo code implementation may look like so:

```js
function yourCheckWinConditionsFunction( state ) {
	if ( /* state is a win or positive end condition */ ) {
		return true; // anything truthy such as 
					 //'true' or a string specifying the reason
	} else {
		return false; // anything falsy such as null or undefined
	}
}
```

### uniqueKey( state )
> Must return a string or number

If your Generate Moves function has a chance of creating duplicate moves and or does not prune repeated moves, implement the *.uniqueKey* function.  A unique key for your state is the absolute minimum information needed to identify a state as unique.  Perhaps it is all of your state or perhaps it is state.key.  This is an optional configuration which can lead to a great performance boost and in some cases double the search depth per time.
```js
function uniqueKey( state ) {
	// somedata and someotherdata together 
	// makes a unique key for this example state
	return state.somedata + state.someotherdata
}
```

# Example

If you have installed this as a npm dependency first change directory to *node_modules/alphabeta/*.

### Tic Tac Toe

Execute the *tic tac toe* example.  Run the example using the command line like so

```bash
# player 1 and 2 both only get 1 look-ahead
node example/tic-tac-toe/index.js 1 1

# player 2 gets 3 look-aheads
node example/tic-tac-toe/index.js 1 3

# player 1 gets 3 look-aheads
node example/tic-tac-toe/index.js 3 1

# player 1 and 2 both only get 9 look-ahead
node example/tic-tac-toe/index.js 9 9
```

### Template

There is an empty template with 'TODO' comments to create a fully working computer vs computer scenario.

```bash
node example/template/index.js
```

### Chomp (from template)

Chomp is a trivial game of two players.  Each player can eat 1, 2, or 3 pieces of a line of 10 pieces long.  The player who eats the last peices wins.  This example uses template/index.js as a boilerplate.

```bash
node example/template/chomp.js
```
### Chase (from template)

Chase is a trivial game of two entities, a preditor and prey.  Each entity can move up to its maximum speed.  There are a lot of win conditions such as the prey getting too far or the preditor running out of energy.  This example uses template/index.js as a boilerplate.

```bash
# get info
node example/template/chase.js

# give the entities nearly no brains and watch the prey win
node example/template/chase.js 1

# but the prey is doomed so long as the preditor plays well
node example/template/chase.js 4

```

# FAQ

**What is the state object?**  It is whatever you decided is best for your problem.  It is what **generateMovesFunction** creates and what **scoreFunction** takes as an argument.  If your problem is tic tac toe, then perhaps state contains the current board and some other data that you find interesting such as what the previous move was.

**Why is there a .step(callback) function and not just .allSteps(callback)?**  Depending on the depthParameter and the average number of moves generated from **generateMovesFunction** calculating the best next state can take a very long time.  Using **.step(callback)** allows you to move the calculation forward towards completion without blocking and exit when you want, such as after 10 seconds.

**What is the difference between a move and a state?** A move is the just the state that comes after some previous state.  It's just another name for state.


# Related AI Projects
This is part of a set of related projects.

* [AlphaBeta](https://www.npmjs.com/package/alphabeta)
* [Boosting](https://www.npmjs.com/package/boosting)
* [GeneticAlgorithm](https://www.npmjs.com/package/geneticalgorithm)
* [NearestNeighbour](https://www.npmjs.com/package/nearestneighbour)
* [NeuralNet](https://www.npmjs.com/package/neuralnet)

# References

* [Instructor: Patrick Winston from MIT](https://www.youtube.com/watch?v=STjW3eH0Cik)
* [Wikipedia entry for Minimax](https://en.wikipedia.org/wiki/Minimax)


[gitter-url]: https://gitter.im/panchishin/alphabeta
[gitter-image]: https://badges.gitter.im/panchishin/alphabeta.png

[downloads-image]: http://img.shields.io/npm/dm/alphabeta.svg
[downloads-url]: https://www.npmjs.com/~panchishin

[travis-url]: https://travis-ci.org/panchishin/alphabeta
[travis-image]: http://img.shields.io/travis/panchishin/alphabeta.svg

[license-image]: https://img.shields.io/badge/license-Unlicense-green.svg
[license-url]: https://tldrlegal.com/license/unlicense


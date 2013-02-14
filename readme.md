# A language where everything is a dictionary of functions

* Lazily evaluated
* Optional type inference
* Stateful
* Literate programming with Markdown
* Partial application

		# comment

		##
		multiline comment
		##

## Assignment

	number: 42
	opposite: true

## Dictionaries and key/value access

Everything is a dictionary.

	city:
		resident:
			name: 'john'

Access values:

	city resident name # 'john'

## conditions:

Use '->'.

	number: opposite -> -42

	number: opposite -> -42 | 42

## functions:

All key/values in the dictionary are actually functions and can take parameters.

	square x: x * x

By default, the dictionary itself (aka 'this' or 'self') is return after being evaluated

	example param:
		wat: param

	example 'hi' # evaluates to (wat: 'hi')

You may mark a specific value to return with '~'

	do_stuff x y:
		one_thing
		another_thing
		~42

	do_stuff 1 2 # evaluates to 42

In contrast

	do_stuff x y: # returns 42
		one_thing
		another_thing
		42

Will return (one_thing, another_thing, 42); in other words, the evaluated dictionary for do_stuff

## anonymous functions

When using colon with argument application, functions are automatically anonymous:

	(1,2,3,4) each x: x+4-2/2

Here, the portion 'x: x+4-2/2' is anonymous

## lists:

	list: 1, 2, 3, 4

equivalent to:

	list:
		1
		2
		3
		4

## modules:

	math:
		root: sqrt
		square x: x * x
		cube: square . square # same as cube x: square (square x)

	math square 12
	math root 12
	math cube 12

# objects:

	Walrus name:
		name: name # setter/getter
		roar: "rawr! I'm {^name}"

	wally: Walrus 'wally'
	wally roar # "rawr! I'm wally"


The '^' operator accesses the parent dictionary.

	Seal:
		name: 'francis'
		set_name n:
			^name: n

	s: Seal
	s name # 'francis'
	s set_name 'george'
	s name # 'george'

# checking existence:

	print (elvis defined? -> "I knew it!")

If elvis is undefined, will evaluate just into the unapplied print dictionary

# list comprehensions:

	cubes: (math cube x where x in list)

# loop structures

'each' (aka map) will apply a function to each element without affecting the list

	list: 1,2,3,4,5
	list each n: n+1

'fold' works similarly, as expected:

	list: 1,2,3,4,5
	list fold () n, ns: ns append n+1

# string interpolation

	"this is a string {this_is_a_key}"
	'this is a raw string. no interpolation'

# default parameters

Place in parens to the right of the parameter.

	fill container ('mug') liquid ('coffee'):
		"Filling the {container} with {liquid}"

## more conditionals

	happy and knows_it -> clap hands | show_it

	date: friday -> sue | jill

## more comprehensions

	eat food where food in ('toast','cheese','wine')

### fine five course dining.

	courses: 'greens', 'caviar', 'truffles', 'roast', 'cake'
	menu i + 1, dish where dish, i in courses

	years_old: (max: 10, ida: 9, tim: 11)
	ages: years_old each child age: "{child} is {age}"

	files:
		filenames each f: compile (f contents to_string)

## list splicing and slicing

	ns: 1,2,3,4,5,6,7,8,9
	start: ns from 0 to 2 # 1,2,3
	middle: ns from 3 to 5 # 4,5,6
	end: ns from 3 # 4,5,6,7,8,9
	copy: ns

None of it modifies the original list

## chaining

If apple's methods return apple, all the following are equivalent:

	apple shine_with 'shirt' eat

	apple
		shine_with 'shirt'
		eat

	apple shine_with 'shirt'
	apple eat

## more complex conditionals

Horizontal conditionals (not very readable):

	grade student: student excellent_work? -> "A+" | (student okay_work? -> (student tried_hard? -> "B-" | "C") | "B")

For complex conditionals the above is not all that readable. As an alternative, we have vertical conditionals:

'else ->' is an alias for '|'

Using elses:

	grade student:
		student excellent_work? -> "A+"
			else student okay_work? ->
				student tried_hard -> "B-"
				else -> "C"
			else -> "B"

More declarative truth tables:

		table (student excellent_work?), (student okay_work?), (student tried_hard?) where
			true,_,_         : "A+"
			false,true,true  : "B"
			false,true,false : "B-"
			false,false,_    : "C"

Could be implemented entirely in the language itself probably

		table conditions:
			cs: conditions
			where structure:
				structure each vals result:
					vals is ^cs -> ~result

# destructuring assignment

	a, b, c: 1, 2, 3

Evaluates into:

	a: 1
	b: 2
	c: 3

# try/catch

	try
		nonexistent / undefined
	catch error
		"And the error is ... {error}"

## Operators

	x is y
	x isnt y
	not x
	x and y
	x nand y
	x xor y
	x or y
	x + y
	x * y

### Existential function

'defined?' can be used to test whether a func is defined

	solipsism: (mind defined?) nand (world defined?) ->  true

### Assignment with function shorthand

You can implicitly apply methods (keys) to new keys

	speed: 0
	(speed defined?): 12    # will not assign since it was defined as 0

	x: 4
	(x +): 4   # x: x + 4

	greet: "hey yo"
	(greet interpolate): '!' #     greet: greet interpolate '!'

	x: 4
	(x +) y: y    # x y: x + y

## Parent classes

	Animal name: # 'name' can be thought of as a constructor parameter
		name: name # getter/setter
		move meters:
			print "{^name} moved {meters as string}m" # '^' can be thought of as 'self'

	# parent classes are keys mapping to an instantiated parent classes
	Snake name:
		parent: Animal name
		move:
			print 'Slithering...'
			^parent move 5

	Horse name:
		parent: Animal name
		move:
			print 'Galloping...'
			^parent move 45

	sam: Snake 'sammy the python'
	tom: Horse 'tommy the palomino'

	# None of the above stuff is actually evaluated until called

	sam move
	tom move

	# Outputs:
	# > Slithering...
	# > sammy the python moved 5m
	# > Slithering...
	# > tommy the palomino moved 45m

### How the last two calls in the above example evaluate:

'*' denotes something we haven't evaluated yet.

Nothing evaluates until actually called with the correct number of parameters

	sam move
	(Snake 'sammy the python') move    # params satisfied for Snake
	(parent: *, move: *) move          # evalute Snake 'sammy'. move satisfied
	(parent: *, move: print "", (Animal 'sammy the python') move 45) move # evaluate move. Animal satisfied
	(parent: *, move: print "", (name: *, move: *) move 45) move # evaluate Animal. move satisified
	(parent: *, move: print "", (name: *, move: print stuff) move 45) move # evaluate Animal. move satisified
		~> (io side-effect) "Slithering..."
	(parent: *, move: print "", io stuff) move # no more calls

### Extending dictionaries

	x:
		y: 1

	a extends x:
		z: 2

	a y # 1

'extends' probably could be implemented in the language itself

Fields can't be set without a setter function (since '^' is only accessible within the dictionary definition itself)

Note: data fields are equivalent to function fields

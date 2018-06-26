# Nanc-in-a-can Canon Generator

Nanc-in-a-can Canon Generator is a series of sc files that can be used to produce temporal canons as the ones created by Conlon Nancarrow. The files are 7 and mostly contain a function each, 2 of them contain `SynthDefs`. The functions `~makeConvCanon` and `~makeDivCanon` are the core of the program, however, two other auxiliary functions have been added to aid the creation of melodies, transposition patterns and tempos. The function `~makeVisualization` generates a visual timeline for each canon and plays it back. The function `~instrument` produces `Pbinds` for each of the canon's voices. 

## Installation
### Using git
`git clone <path-to-folder>`

### Manual download
[Click here](https://github.com/nanc-in-a-can/canon-generator/archive/master.zip) and save the zip file wherever you want.


## Load Project files
Open Supercollider and add and compile the following line of code.

`(path/to/nanc-in-a-can/init.scd").load;`

This starts up the server and loads all the necessary files and functions.

## Examples
Sound Only:

```
(
var melody = ~melodyMaker.pyramidalMelody;
~makeConvCanon.(melody).canon
  .collect(~instrument.(\pianola))
  .do({|synthVoice| synthVoice.play})
)
```
Sound and Visualization:
```
(
var melody = ~melodyMaker.pyramidalMelody;
~makeVisualization.(~makeConvCanon.(melody));
)
```

## API

Explicación de que usamos tipos
Explicación de los Canon, Voice, Melody
```
Canon :: (
  canon: (
    notes: [Float],
    durs: [Float],
    onset: Float,
    bcp: Float,
    cp: Int
  ),
  data: (
    voices: [(transp: Float, tempo: Float )]
  )
)

Note :: (
  durs: Float, 
  notes: [Float] || \rest
)

Melody :: [Note]

Voice :: (tempo: Float, transp: Float)

Voices :: [Voice]
```


### ~makeMelody

Is a function that generates an array of Event objects with durations and notes necessary to create a musical object that might be a simple monody (one pitch value per duration) or a melodic structure with various degrees of density (two or more pitch values per duration) including rests. In case size of the arrays is not the same, the size of the array of events it returns will be equal to the size of the smaller array it took.

#### Type Signature
Takes an array of durations and an array of midi pitch values and returns an array of Event object with the keys `dur` and `note`. 
```
~makeMelody :: ([Float], [Float]) -> Melody
```


#### Example
(
~myMelody= // SIMPLIFICAR EJEMPLO para que se vea pueda copiar el Resultado

~makeMelody.( 
	Array.fill(35, { [4, 8, 16].wchoose([0.2, 0.3, 0.5].normalizeSum).reciprocal } ) 
	,
	Array.fill(35, { [60, 67,[38, 72], 68, 63, 63.5].wchoose([6, 4, 3, 2, 1, 1].normalizeSum) } )
		);

);

~myMelody.postln;

Arguments:

durs_arr. `[Float]`. Duration array. An array of floats that represents in durations in which 1 is equivalent to a bpm of tempo provided in Voices argument of ~makeConvCanon or ~makeDivCanon. Reciprocal of 2, 4, 8, 16 creates half, quarter, eighth and sixteenth notes respectively.  

notes_arr. `[Float] || [[Float]]`. An array of a) floats, b) arrays of floats c) \rest symbols that represents the pitch value(s) in midi notes. If b) is taken then a chord will be returned. If the symbol \rest is taken then a silence value will be returned. The values are midi notes, if floats of midi notes provided then it will produce microtonal values. In the above example 63.5 will produce a E quarter note flat. 

----------------
### ~makeVoices

Similar to makeMelody however it generates tempos and transposition values.

#### Type Signature
Takes an array of tempos and transposition values and returns an array of Event object with the keys `tempo` and `transp`. 
```
~makeVoices :: ([Float], [Float]) -> Voices
```


#### Example

```
(
~myVoices= ~makeVoices.( 
    Array.series(3, 60, 10),
    Array.series(3, -12, 8)
);

~myVoices.postln; // [(transp: -12, tempo: 60), (transp: -4, tempo: 70), (transp: 4, tempo: 80 )]
)
```
#### Arguments:

tempo. An array of floats that generates tempo in bpm for each voice. 

transp. An array of floats that generates a series of transposition values in midi notes. Negative floats will generate descending intervals in relationship to the midi values passed down from melody, positive ones will generate ascending intervales. 

----------
### ~makeConvCanon
A function that generates a convergence-divergence temporal canon using a configuration object defining a convergence point, a melody, and the number of voices (with tempo and transposition).

#### Type Signature
Takes an Event Object with the keys `cp`, `melody` and `voices` and returns a `MadeCanon` (see below for the MadeCanon type definition)
```
~makeConvCanon ::
  (
    cp: Int, 
    melody: Melody
    voices: Voices
  ) -> Canon
```

#### Example
```
~canonConfig = (
  cp: 2,
  melody: [
    (dur: 1, note: 60), 
    (dur: 1, note: 61), 
    (dur: 1, note: 62), 
    (dur: 1, note: 63)
  ],
  voices: [
    (tempo: 70, transp: 0),
    (tempo: 65, transp: -12),
    (tempo: 57, transp: 12),
    (tempo: 43, transp: 8)
  ]
);
~myCanon = ~makeConvCanon.(canonConfig);

~makeVisualization(~myCanon);
```
#### Arguments: 

`~makeConvCanon` takes a single argument, an Event Object with the following keys:

`cp`: `Int`. **Convergence point**. An _integer_ that represents the index of the melodic structure at which all the different voices have the same structural and chronological position simultaneously.  The convergence point might be at any given onset value of the melody.

`melody`: `[(dur: Float, note: [midiNote])]`. A melodic structure represented as an array of Event objects with a duration and a note values. `dur` is float representing rhythmic proportions. `note` may be a midi note -`Int`-, an array of midiNotes -`[Int]`- o the `\rest` keyword `Symbol`. The function `~makeMelody`(#makeMelody) may be used to generate this structure from a pair of arrays.

`voices`: `[(tempo: Float, transp: Int)]`. An array of Event objects with tempo and transposition. The size of the array determines the number of voices of the temporal canon. Tempo is a float that represents a BPM value.  Transp represent a midi note value that will be added to the midi notes established in the melody. Negative numbers are descending intervals, positive numbers are ascending ones. The helper function `~makeVoices` provides an API that allows for the simplified creation of this array, using either tempos or proportions, and generating automatic transpositions relating the temporal propotions to pitch (as proposed by Henry Cowell).

<!-- Inconsistencia entre la ~makeVoices y el modo en el que se construyen las melod�as. Resolver aquello urgentemente.  -->


----------------------
~makeDivCanon

Is a function that generates a divergence-convergence temporal canon based on a musical object that might be a monody or a melodic structure with various degrees of density per attack including rests. 

Arguments:
####
####
#####
#######



----------------------------
### ~makeVisualization.(madeCanon, autoScroll: true) 

#### Type Signature
Takes an Event Object MadeCanon and creates a window object that visualizes and plays back the canon.
```
~makeVisualization :: Canon -> Nil
```

Arguments:

`madeCanon` : `Canon`. A canon of the same type as the one returned by functions such as `~makeConvCanon` or `~makeDivCanon`.

`autoScroll`: `Boolean`.  Default is true.


-----------------------------
### ~canonPreConfigs

A set of canon configurations that function as examples for the Nanc-in-a-Can project. Each melody was design at some point of development of the program and is the outcome of the implementation of one or many ideas concerning the temporal canon concept. 

Configurations:

randomSymmetric4voices. A four voice canon configuration that creates a different melody everytime. The melody is generated using a weighted random process. The pitch set is modal /phrygian depending the octave with microtonal inflections. The durations are based in the theory of harmonic rhythm and the rhythm device of complex denomitar (irrational measures). 

pyramidalMelody. Using the parcials 16 to 40 of a sound with a root in 12 htz we create a pitch configuration that is simultaneously expressed in duration values using the theory of harmonic rhythm. We used the function ~makeMelody to then apply several methods to such an array. The tempo was design proportionally.

Fragment of Study 37 (sonido 13 version)

Fragment of Study 33 

chido_melody

----------------------------
## ~instrument
```
Repeat :: Int
~instrument :: ([Symbol], [Amp, Pan, Out, Repeat) -> ((durs, notes, onset), Int) -> Pbind 
```

`Amp`, `Pan`, `Out` and `Repeat` have default values and are optional.

----------------------------
## synthdef-instrument module.

Contains two sound-generating synthdefs. The first based in an algorithm by James McCartney that emulates a piano-player with the feature that may emulate a limited "prepared" piano timbre. 

----------------------------
## init module.

Loads all the functions and compiles all the synths necessary to operate the Nanc-in-a-Can program. 


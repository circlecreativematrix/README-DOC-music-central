# Description
This is a main spot for instructions on how to run, ideas of what could be added, and ideas of what can be tested. 

# How to Run: 
You first want to look at play.circlecreativematrix.com. At the time of this writing it is still operational ,
- if the website is not operational and you want to continue, update a local [music-central-web](https://github.com/circlecreativematrix/music-central-web) project and serve it using
`python3 -m http.server` or similar.
- Study the format. That is Standard-Note. I
- Standard-Note is a key:value, comma separated format similar to csv, but not quite as it puts the headers into the keys.
- each line is parsed as a separate section. if there are duplicates, right side overwrites left.
- the initial wasm was made using tinygo and [converter-standard-note](https://github.com/circlecreativematrix/converter-standard-note)
  - follow the link for more information on installing and using that one. Look at tests to see the format in action.
- The following addon for chords wasm was made using [transformer-roman-chord   ](https://github.com/circlecreativematrix/transformer-roman-chord) , it runs first, and converts any chords to standard note format.
  - I want to say Augmented `_+` was not working last time I checked. This needs some good test cases. Built in go.

# Alternate way to run:    
WASM is nice in the browser, but so is having a local copy to run everywhere. Please take a look at run.sh in converter-standard-note. 
The original goal was to build all these converters, then have them generate an almost-midi called Note Beat Exchange Format (json notes) , then compile them all together into one big arrangement and output them to midi. 
- [Music-central](https://github.com/circlecreativematrix/music-central) is the main hub of that. I created a find-replace for $variables, and wrote music-central so converters can be found on the filesystem, even in windows.
- the src/config/config.yml paths are very important to get this up and running with  [converter-standard-note](https://github.com/circlecreativematrix/converter-standard-note) and the other converters.
- the paths are double escaped for windows, but need to be forward slashes (/) for mac and linux.
- pytest is funky on system paths. I run it in /tests/ and set my system path to look right above it. If you know how to solve that so pytest runs in the root directory , please feel free to let me know in an issue or comment.
- 

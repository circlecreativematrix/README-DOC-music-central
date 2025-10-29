# Description
This is a main spot for instructions on how to run, ideas of what could be added, and ideas of what can be tested. 

# How to Run: 
You first want to look at [play.circlecreativematrix.com](play.circlecreativematrix.com) . At the time of this writing it is still operational ,
- if the website is not operational and you want to continue, update a local [music-central-web](https://github.com/circlecreativematrix/music-central-web) project and serve it using
`python3 -m http.server` or similar.
- Study the format. That is Standard-Note.
- Standard-Note is a key:value, comma separated format similar to csv, but not quite as it puts the headers into the keys.
- to see what the options are, [look at the source](https://github.com/circlecreativematrix/converter-standard-note/blob/feature-get-variables-working/src/services/standard_to_nbef.go#L279)
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

# Different Converters In Order Of Importance:
# StandardNote
- notes look like:
- ```
  key_type:major,key_note:C4
  note:2,time:P+1/8
  ```
  - time : {Previous Time} + {quantized time fraction}
  - time can also be an absolute number such as `1.234`
  - time can also be {Previous Time} + absolute number such as `1.234`
  - note can be a number such as 0 , 1,2 , the distance away from the key note.
  - note can also be a letter such as C4#, E4@, F2 , where the # is sharp and the `@` is flat, and the number is the octave it is on. 
- see [converter-standard-note](https://github.com/circlecreativematrix/converter-standard-note)
- it can actually describe a midi file precisely line by line , note by note, down to the millisecond.  It still needs programChanges described though.  
- it is a full text notation that can be used as an interim until midi is requested. This is what [https://play.circlecreativematrix.com](https://play.circlecreativematrix.com) uses, then converts to nbef, then to midi and audio on the web browser.
- in the alternative way to run, it sends nbef to the python music central to be encoded into midi or played live in a music yaml file (MAML) file. 
# NBEF 
JSON format, holds most of the data right before MIDI. This is the intermediary step. It goes {whatever format} >> standard note >> nbef >> midi. Sometimes it goes directly to midi, but that makes it harder to parse and figure out. Layers help the overall ecosystem of this. 
## AtomicNote-Py: 
### IntNote 
  - notes look like :
  - ``` 0 1 2 3 4 5 6 7 8 9 88 2 87 1```
  - triggers functions of notes based on the number hashmap you give it. `[ 0 1 2 3 4 5 6 88 8 0 1 2 3 4 5 6 88 2 2 1 0 ]` will play a partial scale in quarter notes, then call the 88 function, in this case it's set-note length to 8th notes,  then plays 8th notes partial scale, then calls the 88 function to set the notes to half notes, then plays E4, D4, C4 , assuming the set-key is C4.
  - it's easy to encode and play with [atomicNote](https://github.com/circlecreativematrix/atomicnote-py/tree/3-16-2024_projects/py)
  - it's hard to read and decode without a middle logging step.
### DoubleNote 
 - like IntNote, but things like rests, notes, etc , are set as flags on the decimal side.
### Note-Beats
  - notes are like `A3#:1/4` , this is my favorite second to Standard Note.

# Extending this library : 
Once you get the standardNote--> nbef --> midi pipeline, you can build whatever you want on top of it to convert down to standard note, then eventually that can get to midi or audio. This makes music tooling building easy and enjoyable :) 
- StandardNote is line based, and can be written by any language that outputs to text or console.

## Example converter: 
This javascript converts text like : 
```
playSnippet("2:1/8 1 0 1 2 2 2:1/4 1:1/8 1 1:1/4 2:1/8 4 4:1/4 2:1/8 1 0 1 2 2 2 2 1 1 2 1 0:1/1", "\n")
```
into StandardNote:
```
key_type:major,key_note:C4
note:2,time:P+1/8
note:1,time:P+1/8
note:0,time:P+1/8
note:1,time:P+1/8
note:2,time:P+1/8
note:2,time:P+1/8
note:2,time:P+1/4
note:1,time:P+1/8
note:1,time:P+1/8
note:1,time:P+1/4
note:2,time:P+1/8
note:4,time:P+1/8
note:4,time:P+1/4
note:2,time:P+1/8
note:1,time:P+1/8
note:0,time:P+1/8
note:1,time:P+1/8
note:2,time:P+1/8
note:2,time:P+1/8
note:2,time:P+1/8
note:2,time:P+1/8
note:1,time:P+1/8
note:1,time:P+1/8
note:2,time:P+1/8
note:1,time:P+1/8
note:0,time:P+1/1
```
- code: 
```javascript
   
    function playSnippet(short, joiner = "<br/>") {
      let currentTime = "1/8"
      const listShort = short.split(" ")
      let returnSegments = []
      returnSegments.push(playHeader("major", "C4"))
      for (item of listShort) {
        let itemList = item.split(":")
        if (itemList.length == 1) {
          returnSegments.push(`note:${itemList[0]},time:P+${currentTime}`)
        }
        if (itemList.length == 2) {
          currentTime = itemList[1]
          returnSegments.push(`note:${itemList[0]},time:P+${currentTime}`)
        }
      }
      return returnSegments.join(joiner)
    }
    function playHeader(type = "major", note = "C4") {
      return `key_type:${type},key_note:${note}`
    }
```
this can then either be plugged into the already working play.circlecreativematrix.com , or be put through the alternate way. 
## further exploration
- the converter-standard-note can have more tests to describe the output nbef.
- the transformer-roman-chord can have more tests to describe the output standardNote.
- Do I have all chord formations? Can I prove that with tests?
  


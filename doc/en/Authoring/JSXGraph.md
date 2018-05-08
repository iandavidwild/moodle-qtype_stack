# JSXGraph

STACK supports inclusion of dynamic graphs using JSXGraph: [http://jsxgraph.uni-bayreuth.de/wiki/](http://jsxgraph.uni-bayreuth.de/wiki/).

Note, we strongly recommend you do not use an HTML aware editor when using JSXGraph questions.  Instead turn off the editor within Moodle and edit the raw HTML.

## Include basic plots.

This example is based on the documentation for [curve](http://jsxgraph.uni-bayreuth.de/wiki/index.php/Curve) and the [even simpler function plotter](http://jsxgraph.uni-bayreuth.de/wiki/index.php/Even_simpler_function_plotter) example.

To include a basic dynamically generated sketch into a STACK question, first define the expression of the graph to be plotted in the question variables.  For example

    a:rand(6)-3
    fx:sin(x)+a

Then include the following question text, which includes a simple `[[jsxgraph]]` block.  In particular note the lack of `<script>` tags which you might expect to include.

    <p>Type in an algebraic expression which has the graph shown below.</p>
    [[jsxgraph]]
      // boundingbox:[left, top, right, bottom]
      var board = JXG.JSXGraph.initBoard(divid, {boundingbox: [-10, 5, 10, -5], axis: true});
      var f = board.jc.snippet('{#fx#}', true, 'x', true);
      board.create('functiongraph', [f,-10,10]);
    [[/jsxgraph]]
    <p>\(f(x)=\) [[input:ans1]] [[validation:ans1]]</p>

Note the code `board.jc.snippet('{#fx#}', true, 'x', true);` which turns a reasonable expression for a function into the Javascript function.  You cannot just plot the `functiongraph` on its own.

To make a working question, you will then need to add in `fx` as the model answer to input `ans1`, a question note (e.g. `\({@fx@}\)`) and an appropriate potential response tree.

## Interactive elements

In this example define the question variables as

    fx:int(expand((x-1)*(x+1)*(x-2)),x)

This question contains an interactive sliding element.

    <p>A graph, together with the tangent line and its slope, are shown below.  Find an algebraic expression for the graph shown below.</p>
    [[jsxgraph]]
      // boundingbox:[left, top, right, bottom]
      var board = JXG.JSXGraph.initBoard(divid, {boundingbox: [-5, 10, 5, -10], axis: true});
      var f = board.jc.snippet('{#fx#}', true, 'x', true);
      curve = board.create('functiongraph', [f,-10,10], {strokeWidth:2});
      dcurve = board.create('functiongraph', [JXG.Math.Numerics.D(f),-10,10], {strokeColor:'#ff0000', strokeWidth:1, dash:2});
      var p = board.create('glider',[1,0,curve], {name:'Drag me'});
      board.create('tangent',[p], {name:'Drag me'});
      var q = board.create('point', [function(){return p.X();}, function(){return JXG.Math.Numerics.D(f)(p.X());} ], {withLabel:false});
      board.unsuspendUpdate();
    [[/jsxgraph]]
    <p>\(f(x)=\) [[input:ans1]] [[validation:ans1]]</p>

In this example the student can interact with a dynamic diagram to help them understand what is going on.

## An example with a slider

In this example we provide a simple slider.  Notice in this example we use the Javascript notation `a**x` for \(a^x\) and not Maxima's `a^x`.

    [[jsxgraph]]
      // boundingbox:[left, top, right, bottom]
      var board = JXG.JSXGraph.initBoard(divid, {boundingbox: [-5, 10, 5, -10], axis: true});
      var a = board.create('slider',[[-3,6],[2,6],[0,2,6]],{name:'a'}); 
      curve = board.create('functiongraph', [function(x) {return a.Value()**x}], {strokeWidth:2});
      board.unsuspendUpdate();
    [[/jsxgraph]]



## General considerations when building interactive graphs

In general you should pay attenttion on how your graph reacts to the student returning to the page/question later i.e. will your graph 
reset to display the original situation or will it atleast move all movable things to the positions the student last left them and if 
the student can do things that are not actually considered as part of the answer e.g. zoom out or pan the view do you also remember 
those actions. If your graph is not used for inputting answers then this is not a major issue but if it is then you will need to solve 
this issue. Basically, storing the state of the interactive graph is a key thing that the author of that graph needs to deal with.

The basic structure of such an graphs logic is as follows:

 1. Load existing state or if not found initialise with defaults.
 2. Draw the graph based on that state.
 3. Attach listeners to everything that can be changed in the graph and store those changes into the state in those listeners.

The simplest solution for storing state is to add an String type input field to the question. That input field should not be connected 
to any PRTs and you should turn off the validation and verification of the field. You can even use the syntax hint feature to pass in a 
default value but only if that is not parametric. You can use that input field to store the state of the graph as a string, for example 
as a JSON encoded structure. For example like this, assuming the name of the String input is named "stateStore":


    [[jsxgraph input-ref-stateStore="stateRef"]]
      // Note that the input-ref-X attribute above will store the element identifier of the input X in 
      // a variable named in the attribute, you can have multiple references to multiple inputs.

      // Create a board like normal.
      var board = JXG.JSXGraph.initBoard(divid, {axis: true});

      // State represented as an JS-object, first define default then try loading the stored
      var state = {'x':4, 'y':3};
      var stateInput = document.getElementById(stateRef);
      if (stateInput.value && stateInput.value != '') {
        state = JSON.parse(stateInput.value);
      }

      // Then make the graph represent the state
      var p = board.create('point',[state['x'],state['y']]);

      // And finally the most important thing, update the stored state when things change
      p.on('drag', function() {
        var newState = {'x':p.X(), 'y':p.Y()};
        // Encode the state as JSON for storage and store it
        stateInput.value = JSON.stringify(newState);
      });

      // As a side note, you typically do not want the state storing input to be directly visible to the user
      // although it may be handy during development to see what happens in it. You might hide it like this:
      stateInput.style.display = 'none';
    [[/jsxgraph]]


In that trivial example you only have one point that you can drag around but that points position will be stored and it will be where 
you left it when you return to the page. However, the position has been stored in a String encoded in JSON format and cannot directly be 
used in STACK side logic. The JSON format is however very handy if you create objects to store dynamically and want to represent things 
of more complex nature but in this example we could have just as well have had two separate Numeric inputs storing just the raw 'x' 
and 'y' coordinates separately as raw numbers and in that case we could have used them directly in STACKs grading logic.

If needed JSON is not impossible to parse in STACK but it is not easy like in JavaScript, mainly because Maxima has no map 
data-structures and is not object oriented. In any case the JSON string generated in the previous example would look like this:

    stateStore:"{\"x\":4,\"y\":3}";

To parse and manipulate it you can use STACKs custom JSON parsing functions:

    tmp:stackjson_parse(stateStore); /* This returns a STACK-map: [stack_map, [x, 4], [y, 3]] */
    x:stackmap_get(tmp,"x");         /* 4 */
    y:stackmap_get(tmp,"y");         /* 3 */
    tmp:stackmap_set(tmp,"z",x*y);   /* [stack_map, [x, 4], [y, 3], [z, 12]] */
    json:stackjson_stringify(tmp);   /* "{\"x\":4,\"y\":3,\"z\":12}" */
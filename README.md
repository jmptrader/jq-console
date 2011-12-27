#jq-console  
  
A jQuery terminal plugin written in CoffeeScript.  
  
This project was spawned because of our need for a simple web terminal plugin   
for the <a href="http://repl.it">repl.it</a> project. It tries to simulate a low level terminal by providing (almost)  
raw input/output streams as well as input and output states.  
  
Version 2.0 adds baked-in support for rich multi-line prompting and operation  
queueing.  
  
NOTE: This info is for jq-console v2.0. For jq-console v1.0 see README-v1.md.  
  
  
##Tested Browsers  
  
The plugin has been tested on the following browsers:  
  
* IE 9-10
* Chrome 10-14
* Firefox 3.6-6
* Opera 11  
  
  
##Getting Started  
  
###Echo example

```css
    /* The console container element */
    #console {
      position: absolute;
      width: 400px;
      height: 500px;
      background-color:black;
    }
    /* The inner console element. */
    .jqconsole {
        padding: 10px;
    }
    /* The cursor. */
    .jqconsole-cursor {
        background-color: gray;
    }
    /* The cursor color when the console looses focus. */
    .jqconsole-blurred .jqconsole-cursor {
        background-color: #666;
    }
    /* The current prompt text color */
    .jqconsole-prompt {
        color: #0d0;
    }
    /* The command history */
    .jqconsole-old-prompt {
        color: #0b0;
        font-weight: normal;
    }
    /* The text color when in input mode. */
    .jqconsole-input {
        color: #dd0;
    }
    /* Previously entered input. */
    .jqconsole-old-input {
        color: #bb0;
        font-weight: normal;
    }
    /* The text color of the output. */
    .jqconsole-output {
        color: white;
    }
```

```html
    <div id="console"></div>
    <script src="jquery.js" type="text/javascript" charset="utf-8"></script>
    <script src="jqconsole.js" type="text/javascript" charset="utf-8"></script>
    <script>
      $(function () {
        var jqconsole = $('#console').jqconsole('Hi\n', '>>>');
        var startPrompt = function () {
          // Start the prompt with history enabled.
          jqconsole.Prompt(true, function (input) {
            // Output input with the class jqconsole-output.
            jqconsole.Write(input + '\n', 'jqconsole-output');
            // Restart the prompt.
            startPrompt();
          });
        };
        startPrompt();
      });
    </script>
```
  
###Instantiating  

```javascript
    $(div).jqconsole(welcomeString, promptLabel, continueLabel);
```

* `div` is the div element or selector. Note that this element must be
  explicity sized and positioned `absolute` or `relative`.
* `welcomeString` is the string to be shown when the terminal is first rendered.  
* `promptLabel` is the label to be shown before the input when using Prompt().  
* `continueLabel` is the label to be shown before the continued lines of the  
  input when using Prompt().

###Configuration  
  
There isn't much initial configuration needed, because the user must supply  
options and callbacks with each state change. There are a few config methods  
provided to create custom shortcuts and change indentation width:  
  
* `jqconsole.RegisterShortcut`: Registers a callback for a keyboard shortcut.  
  Takes two arguments:  
  
    * `(int|string) keyCode`: The code of the key pressing which (when Ctrl is  
      held) will trigger this shortcut. If a string is provided, the ASCII code  
      of the first character is taken.  
  
    * `function callback`: A function called when the shortcut is pressed;  
      "this" will point to the JQConsole object.  
  
    Example:  
  
        // Ctrl+R: resets the console.  
        jqconsole.RegisterShortCut('R', function() {  
          this.Reset();  
        });  
  
* `jqconsole.SetIndentWidth`: Sets the number of spaces inserted when indenting  
  and removed when unindenting. Takes one argument:  
  
    * `int width`: The code of the key pressing which (when Ctrl is held) will  
      trigger this shortcut.  
  
    Example:  
  
        // Sets the indent width to 4 spaces.  
        jqconsole.SetIndentWidth(4);  
  
  
##Usage  
  
Unlike most terminal plugins, jq-console gives you complete low-level control  
over the execution; you have to call the appropriate methods to start input  
or output:  
  
* `jqconsole.Input`: Asks user for input. If another input or prompt operation  
  is currently underway, the new input operation is enqueued and will be called  
  when the current operation and all previously enqueued operations finish.  
  Takes one argument:  
  
    * `function input_callback`: A function called with the user's input when  
      the user presses Enter and the input operation is complete.  
  
    Example:  
  
        // Echo the input.  
        jqconsole.Input(function(input) {  
          jqconsole.Write(input);  
        });  
  
* `jqconsole.Prompt`: Asks user for input. If another input or prompt operation  
  is currently underway, the new prompt operation is enqueued and will be called  
  when the current operation and all previously enqueued operations finish.  
  Takes three arguments:  
  
    * `bool history_enabled`: Whether this input should use history. If true,  
      the user can select the input from history, and their input will also be  
      added as a new history item.  
  
    * `function result_callback`: A function called with the user's input when  
      the user presses Enter and the prompt operation is complete.  
  
    * `function multiline_callback`: If specified, this function is called when  
      the user presses Enter to check whether the input should continue to the  
      next line. The function must return one of the following values:  
  
        * `false`: the input operation is completed.  
  
        * `0`: the input continues to the next line with the current indent.  
  
        * `N` (int): the input continues to the next line, and the current  
          indent is adjusted by `N`, e.g. `-2` to unindent two levels.  
      
    * `bool async_multiline`: Whether the multiline callback function should  
      be treated as an asynchronous operation and be passed a continuation  
      function that should be called with one of the return values mentioned  
      above: `false`/`0`/`N`.  
        
    Example:  
  
        jqconsole.Prompt(true, function(input) {  
          // Alert the user with the command.  
          alert(input);  
        }, function (input) {  
          // Continue if the last character is a backslash.  
          return /\\$/.test(input);  
        });  
  
* `jqconsole.AbortPrompt`: Aborts the current prompt operation and returns to  
  output mode or the next queued input/prompt operation. Takes no arguments.  
  
    Example:  
  
        jqconsole.Prompt(true, function(input) {  
          alert(input);  
        });  
        // Give the user 2 seconds to enter the command.  
        setTimeout(function() {  
          jqconsole.AbortPrompt();  
        }, 2000);  
  
* `jqconsole.Write`: Writes the given text to the console in a `<span>`, with an   
  optional class. If a prompt is currently being shown, the text is inserted  
  before it. Takes two arguments:  
  
    * `string text`: The text to write.  
  
    * `string cls`: The class to give the span containing the text. Optional.  
  
    * `bool escape`: Whether the text to write should be html escaped.  
      Optional, defaults to true.  
  
    Examples:  
  
        jqconsole.Write(output, 'my-output-class')  
        jqconsole.Write(err.message, 'my-error-class')  
  
* `jqconsole.SetPromptText`: Sets the text currently in the input prompt. Takes  
  one parameter:  
  
    * `string text`: The text to put in the prompt.  
  
    Examples:  
  
        jqconsole.SetPromptText('ls')  
        jqconsole.SetPromptText('print [i ** 2 for i in range(10)]')  
  
* `jqconsole.ClearPromptText`: Clears all the text currently in the input  
  prompt. Takes one parameter:  
  
    * `bool clear_label`: If specified and true, also clears the main prompt  
      label (e.g. ">>>").  
  
    Example:  
  
        jqconsole.ClearPromptText()  
  
* `jqconsole.GetPromptText`: Returns the contents of the prompt. Takes one  
  parameter:  
  
    * `bool full`: If specified and true, also includes the prompt labels  
      (e.g. ">>>").  
  
    Examples:  
  
        var currentCommand = jqconsole.GetPromptText()  
        var logEntry = jqconsole.GetPromptText(true)  
  
* `jqconsole.Reset`: Resets the console to its initial state, cancelling all  
  current and pending operations. Takes no parameters.  
  
    Example:  
  
        jqconsole.Reset()  
  
* `jqconsole.GetColumn`: Returns the 0-based number of the column on which the  
  cursor currently is. Takes no parameters.  
  
    Example:  
  
        // Show the current line and column in a status area.  
        $('#status').text(jqconsole.GetLine() + ', ' + jqconsole.GetColumn())  
  
* `jqconsole.GetLine`: Returns the 0-based number of the line on which the  
  cursor currently is. Takes no parameters.  
  
    Example:  
  
        // Show the current line and column in a status area.  
        $('#status').text(jqconsole.GetLine() + ', ' + jqconsole.GetColumn())  
  
* `jqconsole.Focus`: Forces the focus onto the console so events can be  
  captured. Takes no parameters.  
  
    Example:  
  
        // Redirect focus to the console whenever the user clicks anywhere.  
        $(window).click(function() {  
          jqconsole.Focus();  
        })  
  
* `jqconsole.GetIndentWidth`: Returns the number of spaces inserted when  
  indenting. Takes no parameters.  
  
    Example:  
  
        jqconsole.SetIndentWidth(4);  
        console.assert(jqconsole.GetIndentWidth() == 4);  
  
* `jqconsole.RegisterMatching`: Registers an opening and closing characters to   
  match and wraps each of the opening and closing characters with a span with   
  the specified class. Takes one parameters:  
      
    * `char open`: The opening character of a "block".  
    * `char close`: The closing character of a "block".  
    * `string class`: The css class that is applied to the matched characters.  
  
    Example:  
  
        jqconsole.RegisterMatching('{', '}', 'brackets');  
  
* `jqconsole.UnRegisterMatching`: Deletes a certain matching settings set by  
  `jqconsole.RegisterMatching`. Takes two paramaters:  
  
    * `char open`: The opening character of a "block".  
    * `char close`: The closing character of a "block".  
  
    Example:  
  
        jqconsole.UnRegisterMatching('{', '}');  
  
* `jqconsole.Dump`: Returns the text content of the console.    
  
* `jqconsole.GetState`: Returns the current state of the console. Could be one  
    of the following:    
      
    * Input: `input`    
    * Output: `output`  
    * Prompt: `prompt`  
  
  Example:
  
        jqconsole.GetState(); //output
    
* `jqconsole.MoveToStart`: Moves the cursor to the start of the current line.   
  Takes one parameter:  
      
    * `bool all_lines`: If true moves the cursor to the beginning of the first  
    line in the current prompt. Defaults to false.  
  
  Example:
  
        // Move to line start Ctrl+A.
        jqconsole.RegisterShortcut('A', function() {
          jqconsole.MoveToStart();
          handler();
        });
    
* `jqconsole.MoveToEnd`: Moves the cursor to the end of the current line.  
  Takes one parameter:  
    
    * `bool all_lines`: If true moves the cursor to the end of the first  
    line in the current prompt. Defaults to false.  
  
  Example:
        
        // Move to line end Ctrl+E.
        jqconsole.RegisterShortcut('E', function() {
          jqconsole.MoveToEnd();
          handler();
        });

* `jqconsole.Disable`: Disables input and focus on the console.

* `jqconsole.Enable`: Enables input and focus on the console.

* `jqconsole.IsDisabled`: Returns true if the console is disabled.

* `jqconsole.ResetHistory`: Resets the console history.

* `jqconsoe.ResetMatchings`: Resets the character matching configuration.

* `jqconsole.ResetShortcuts`: Resets the shortcut configuration.

##Default Key Config  
  
The console responds to the followind keys and key combinations by default:  
  
* `Delete`: Delete the following character.  
* `Ctrl+Delete`: Delete the following word.  
* `Backspace`: Delete the preceding character.  
* `Ctrl+Backspace`: Delete the preceding word.  
* `Ctrl+Left`: Move one word to the left.  
* `Ctrl+Right`: Move one word to the right.  
* `Home`: Move to the beginning of the current line.  
* `Ctrl+Home`: Move to the beginnig of the first line.  
* `End`: Move to the end of the current line.  
* `Ctrl+End`: Move to the end of the last line.  
* `Shift+Up`, `Ctrl+Up`: Move cursor to the line above the current one.  
* `Shift+Down`, `Ctrl+Down`: Move cursor to the line below the current one.  
* `Tab`: Indent.  
* `Shift+Tab`: Unindent.  
* `Up`: Previous history item.  
* `Down`: Next history item.  
* `Enter`: Finish input/prompt operation. See Input() and Prompt() for details.  
* `Shift+Enter`: New line.  
* `Page Up`: Scroll console one page up.  
* `Page Down`: Scroll console one page down.  
  
##ANSI escape code SGR support

jq-console implements a large subset of the ANSI escape code graphics.  
Using the `.Write` method you could add style to the console using  
the following syntax: 

`ASCII 27 (decimal) or 0x1b (hex)`  `[`  `Graphics code``m`

Example:

  jqconsole.Write('\033[31mRed Text');

You'll need to include the `ansi.css` file for default effects or create your  
own using the css classes from the table below.

<table>
  <tr>
    <th>Code</th>
    <th>Effect</th>
    <th>Class</th>
  </tr>
  <tr>
    <td>0</td>
    <td>Reset / Normal</td>
    <td></td>
  </tr>
  <tr>
    <td>1</td>
    <td>Bold</td>
    <td>jqconsole-ansi-bold</td>
  </tr>
  <tr>
    <td>2</td>
    <td>Faint</td>
    <td>jqconsole-ansi-lighter</td>
  </tr>
  <tr>
    <td>3</td>
    <td>Italic</td>
    <td>jqconsole-ansi-italic</td>
  </tr>
  <tr>
    <td>4</td>
    <td>Underline</td>
    <td>jqconsole-ansi-underline</td>
  </tr>
  <tr>
    <td>5</td>
    <td>Blink: 1s delay</td>
    <td>jqconsole-ansi-blink</td>
  </tr>
  <tr>
    <td>6</td>
    <td>Blink: 0.5s delay</td>
    <td>jqconsole-ansi-blink-rapid</td>
  </tr>
  <tr>
    <td>8</td>
    <td>Hidden</td>
    <td>jqconsole-ansi-hidden</td>
  </tr>
  <tr>
    <td>9</td>
    <td>Crossed-out</td>
    <td>jqconsole-ansi-line-through</td>
  </tr>
  <tr>
    <td>10</td>
    <td>Remove all fonts</td>
    <td></td>
  </tr>
  <tr>
    <td>11-19</td>
    <td>Add custom font</td>
    <td>jqconsole-ansi-fonts-n where n is code-10</td>
  </tr>
  <tr>
    <td>20</td>
    <td>Add Fraktur font (not implemented in ansi.css)</td>
    <td>jqconsole-ansi-fraktur</td>
  </tr>
  <tr>
    <td>21</td>
    <td>Remove Bold and Faint effects</td>
    <td></td>
  </tr>
  <tr>
    <td>22</td>
    <td>Same as 21</td>
    <td></td>
  </tr>
  <tr>
    <td>23</td>
    <td>Remove italic and fraktur effects</td>
    <td></td>
  </tr>
  <tr>
    <td>24</td>
    <td>Add underline text decoration.</td>
    <td>jqconsole-ansi-underline</td>
  </tr>
  <tr>
    <td>25</td>
    <td>Rmove blinking effect(s).</td>
    <td></td>
  </tr>
  <tr>
    <td>28</td>
    <td>Reveal text</td>
    <td></td>
  </tr>
  <tr>
    <td>29</td>
    <td>Remove cross-out text effect</td>
    <td></td>
  </tr>
  <tr>
    <td>30-37</td>
    <td>Set foreground color to color from the color table below</td>
    <td>jqconsole-ansi-color-{COLOR} where {COLOR} is the color name</td>
  </tr>
  <tr>
    <td>39</td>
    <td>Restore default foreground color</td>
    <td></td>
  </tr>
  <tr>
    <td>40-47</td>
    <td>Set background color to color from the color table below</td>
    <td>jqconsole-ansi-background-color-{COLOR} where {COLOR} is the color name</td>
  </tr>
  <tr>
    <td>49</td>
    <td>Restore default background color</td>
    <td></td>
  </tr>
  <tr>
    <td>51</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>53</td>
    <td>Adds overline effect to text</td>
    <td>jqconsole-ansi-overline</td>
  </tr>
  <tr>
    <td>54</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>55</td>
    <td>Remove overline effect</td>
    <td></td>
  </tr>
</table>

##CSS Classes  
  
Several CSS classes are provided to help stylize the console:  
  
* `jqconsole`: The main console container.  
* `jqconsole, jqconsole-blurred`: The main console container, when not in focus.  
* `jqconsole-cursor`: The cursor.  
* `jqconsole-header`: The welcome message at the top of the console.  
* `jqconsole-input`: The prompt area during input. May have multiple lines.  
* `jqconsole-old-input`: Previously-entered inputs.  
* `jqconsole-prompt`: The prompt area during prompting. May have multiple lines.  
* `jqconsole-old-prompt`: Previously-entered prompts.  
* `jqconsole-composition`: The div encapsulating the composition of multi-byte  
    characters.

  
Of course, custom classes may be specified when using `jqconsole.Write()` for  
further customization.  
  
  
##Contributors  
  
[Max Shawabkeh](http://max99x.com/)    
[Amjad Masad](http://twitter.com/amjad_masad)  

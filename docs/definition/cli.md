# CLI
Our command line interface makes creating and editing Yuzu projects' blocks and pages far easier and can be found here: [Yuzu Definition CLI repositiory](https://github.com/balanced-dev/yuzu-definition-cli)

---

# Options
| Options        				| Alias 			| Description                                           |
| ----------------------------- | -----------------	| -----------------------------------------------------	|
| --version                     | -V				| Output the version number                             |
| --help             			| -h				| Output usage information (list of options & commands) |

---

# Commands
| Command  | Alias | Parameters    	                                     | Description                                                                                                   | Example(s)                                                    |
| -------- | ----- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| config   |       | &lt;type&gt;				                         | Shows the current applied settings                                                                            |                                                               |
| create   | c	   | &lt;name&gt;				                         | Generate the definition side of a project with basic configuration                                            | `create "Example Project"`                                    |
| add      | a     | &lt;type&gt; &lt;name&gt; [area]				     | Add a new block or page (optionally to an existing directory)                                                 | `add block testBlock contentBlocks`, `add page examplePage`   |
| addState | as    | &lt;type&gt; &lt;name&gt; &lt;state&gt; [area]      | Add new data state for an existing block or page                                                              | `addState block formButton longTitle _forms/_formElements`    |
| rename   | r	   | &lt;type&gt; &lt;oldName&gt; &lt;newName&gt; [area] | Rename a block or page                                                                                        | `rename page index homePage`                                  |

!> All commands (apart from `config` and `create`) should be run in the `definition.src` directory of a project
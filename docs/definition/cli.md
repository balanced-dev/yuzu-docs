# CLI
Our command line interface makes creating and edition projects' Yuzu Definitions far easier, and can be found here: [Yuzu Definition CLI repositiory](https://github.com/balanced-dev/yuzu-definition-cli)

---

# Options
| Options        				| Alias 			| Description                                           |
| ----------------------------- | -----------------	| -----------------------------------------------------	|
| --version                     | -V				| Output the version number                             |
| --help             			| -h				| Output usage information (list of options & commands) |

---

# Commands
| Command        				| Alias 			| Parameters    	                | Description                                                           | Example(s)                                                    |
| ----------------------------- | -----------------	| --------------------------------- | ---------------------------------------------------------------------	| -------------------------------------------------------------	|
| config                        |   				| <type>				            | Shows the current applied settings                                    |                                                               |
| create                        | c					| <name>				            | Generate the definition side of a project with basic configuration    | `create "Example Project"`                                    |
| add                           | a 				| <type> <name> [area]				| Add a new block or page (optionally to an existing directory)         | `add testBlock block contentBlocks`, `add examplePage page`   |
| addState                      | as				| <type> <name> <state> [area]      | Add new data state for an existing block or page                      | `addState block formButton longTitle _forms/_formElements`    |
| rename                        | r					| <type> <oldName> <newName> [area] | Rename a block or page                                                | `rename page hmoePage homePage`                               |

!> All commands (apart from `config` and `create`) should be run in the `definition.src` directory of a project

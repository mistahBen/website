
# The two replace variables pattern

## The main replace variable pattern

Here using a **form extension with the main replace statement**
```yaml
  - trigger: ":form"
    replace: "Hey {{form1.name}}, how are you ?" 
    # here form1.name because the variable name comes from the variable form1, 
    # so we need to retrieve it through the form1.name statement.
    vars:
      - name: "form1"
        type: form
        params:
          layout: "Hey [[name]], how are you?"
```


## The replace form shorthand pattern

Here using the **main replace statement form extension shorthand**
```yaml
  - trigger: ":form"
    form: "Hey [[name]], how are you?"
```



# Variable Extensions


## Single Extensions Use Cases

General Structure of an Extension Configuration
```yaml
#  - trigger: ":now"
#    replace: "It's {{mytime}}"
#    vars:
      - name: VARIABLE_NAME
        type: EXTENSION_TYPE
        params:
          ATTR: VALUE
          ATTR2: VALUE
          
      - name: VARIABLE_NAME_2
        type: EXTENSION_TYPE
        params:
          ATTR: VALUE
          ATTR2: VALUE

	  ...
``` 

Structure of a Nested Extension Configuration
```yaml
#  - trigger: ":now"
#    replace: "It's {{mytime}}"
#    vars:
      - name: VARIABLE_NAME
        type: EXTENSION_TYPE
        params:
          ATTR: VALUE
          ATTR2: VALUE
          
      - name: VARIABLE_NAME_2
        type: form
        params: ...
        
``` 

Optional common attributes
```yaml
#  - trigger: ":now"
#    replace: "It's {{mytime}}"
#    vars:
      - name: VARIABLE_NAME
        type: EXTENSION_TYPE
        inject_vars: BOOLEAN # default = true
        params:
          ATTR: VALUE
          ATTR2: VALUE
		  depends_on: ["one"]
		  
``` 

Common variables `vars` components : 
- `name` : variable name **(mandatory)**
- `type` : variable type for global extensions **(mandatory)**
- `params` : variable parameters (common but **optional**)
- `inject_vars` : Let user specify if variables names present in the variable params should be injected or just treated as a `string` (default set to `true` : **optional**).

Optional variable parameters `params` :
- `depends_on` : list of variable names after which this one should be instantiated (default order : global_variables -> local_variables in order of declaration)

___
### Compute Extensions

#### Date Extension
```yaml
  - trigger: ":now"
    replace: "It's {{mytime}}"
    vars:
      - name: mytime
        type: date
        params:
          format: "%H:%M"
          offset: 86400
          locale: "en-US"
``` 

Possible Variable Injections :
- `format` 
- `offset`
- `locale`


`type: date`
`params:`
	`format:` optional
	`offset:` optional
	`locale:` optional

#### Random Extension
```yaml
  - trigger: ":quote"
    replace: "{{output}}"
    vars:
      - name: output
        type: random
        params:
          choices:
            - "Every moment is a fresh beginning."
            - "Everything you can imagine is real."
            - "Whatever you do, do it well."
```

Possible Variable Injections :

- `choices` 

`type: random`
`params:`
	`choices:`
		`- ...`
		`- ...`
	
> [!INFO] possible `choices` injected values :
>
> `\n` seperated string : `"string\nvar\nthree"` 
> 
> `|` YAML pipe : 
> ```yaml
> echo: |
>   "arr"
>   "ray"
>   "var"
> ```
> 

> [!WARNING] impossible `choices` directly injected values :
> 
> NOK : `"['arr','var']"` # 
> NOK : `["arr", "ray", "var"]` # 
> NOK : `'["arr", "ray", "var"]'` # 
> NOK : `"\\['arr','var'\\]"` # 
> NOK : `"\['arr','var'\]"` # 
> NOK : `"['arr','var']"` (seen as string)


### External Resources Extensions


#### Echo Extension
```yaml
  - trigger: ":greet"
    replace: "Hello {{myname}}"
    vars:
      - name: myname 
        type: echo 
        params:
          echo: "John"
```
Possible Variable Injections :
- `echo`

`type: echo`
`params:`
	`echo:`


#### Shell Extension
```yaml
  - trigger: ":ip"
    replace: "{{output}}"
    vars:
      - name: output
        type: shell
        params:
          cmd: "curl 'https://api.ipify.org'"
          shell: wsl
          trim: false
          debug: true
```
Possible Variable Injections :
- `cmd` : (mandatory) (direct injection OK)
- `shell` : (optional) default : `Powershell` in Windows 10, enums : `wsl` (windows), `bash` (linux/MacOS), `command prompt` (windows). (direct injection OK)
- `trim` : (optional) (direct injection OK)
- `debug` : (optional) (direct injection OK)

`type: shell`
`params:`
	`cmd:`
	`shell:` optional
	`trim:` optional
	`debug:` optional


#### Script Extension
```yaml
  - trigger: ":pyscript"
    replace: "{{output}}"
    vars:
      - name: output
        type: script
        params:
          args:
            - python
            - /path/to/your/script.py
```
Possible Variable Injections :
- `args`

`type: script`
`params:`
	`args:`
		`- ...`
		`- ...`

> [!WARNING] Impossible direct variable injection methods
> 
> 
> ```yaml
>       echo: |
>         - python
>         - ./.espanso/scripts/no_env/script.py
> 
>       echo:
>         python
>         ./.espanso/scripts/no_env/script.py
> 
>       echo:
>         - python
>         - ./.espanso/scripts/no_env/script.py
> 
>       echo: |
>         "python"
>         "./.espanso/scripts/no_env/script.py"
>         
>       echo: "python\n./.espanso/scripts/no_env/script.py"
> ```

> [!SUCCESS] Can only be directly injected one string at a time in the args list
> 
> ```yaml
> global_vars:
>   - name: echoedcomfrag
>     type: echo
>     params:
>       echo: "./.espanso/scripts/no_env/script.py"
> 
> matches:  
>   - trigger: ":pyscript"
>     replace: "{{output}}"
>     vars:
>       - name: output
>         type: script
>         params:
>           args:
>             - python
>             - "{{echoedcomfrag}}"
> ```
> 

> [!INFO] Not tested
> 
> - from local scope list output (like with shell output) 
> - from local scope list definition

### Configuration Extensions

#### Match Extension
```yaml
  - trigger: ":one"
    replace: "nested"

  - trigger: ":nested"
    replace: "This is a {{output}} match"
    vars:
      - name: output
        type: match
        params:
          trigger: ":one"
```

NO possible Variable Injections :
- `trigger` -> Can't be injected, produce "missingsub match"

`type: match`
`params:`
	`trigger:`



### Variable Configuration Type

#### Global Extension

`type: global` is used to tell Espanso to handle the variable as a reference to a global variable

```yaml
global_vars:
  - name: three
    type: shell
    params:
      cmd: "echo three"

matches:
  - trigger: ":hello"
    replace: "hello {{one}} {{two}} {{three}}"
    vars:
      - name: one
        type: shell
        params:
          cmd: "echo one"
      - name: two
        type: shell
        params:
          cmd: "echo two"
      - name: three
        type: global
```

```yaml
      - name: three
        type: global
```


### User Input Extensions


#### Clipboard Extension
```yaml
  - trigger: ":a"
    replace: "<a href='{{clipboard}}' /></a>"
    vars:
      - name: "clipboard"
        type: "clipboard"
```

NO possible Variable Injections `*`
`*`_ Clipboard can be made as a variable if we use a command to tell the OS to copy something to clipboard  



#### Choice Extension

```yaml
  - trigger: ":quote"
    replace: "{{output}}"
    vars:
      - name: output
        type: choice
        params:
          values:
            - "Every moment is a fresh beginning."
            - "Everything you can imagine is real."
            - "Whatever you do, do it well."
```


```yaml
matches:
  - trigger: ":quote"
    replace: "{{output}}"
    vars:
      - name: output
        type: choice
        params:
          values:
            - label: "Every moment is a fresh beginning."
              id: "bar"
            - label: "Everything you can imagine is real."
              id: "foo"
            - label: "Whatever you do, do it well."
              id: "foobar"
```

`type: choice`
`params:`
  `- values:`
	  `- "value one"`
	  `- "value two"`
	  `- "value three"`

`type: choice`
`params:`
  `- values:`
	`- label:`
		`id:` 
	`- label:`
		`id:` 

#### Form Extension
```yaml
  - trigger: ":form"
    replace: "Hey {{form1.name}}, how are you ?" 
    # here form1.name because the variable name comes from the variable form1, 
    # so we need to retrieve it through the form1.name statement.
    vars:
      - name: "form1"
        type: form
        params:
          layout: "Hey [[name]], how are you?"
```


```yaml
  - trigger: ":form"
    replace: "Hey {{form1.name}}, how are you? Do you like {{form1.fruit}}?"
    vars:
      - name: "form1"
        type: form
        params:
          layout: "Name: [[name]] \nFruit: [[fruit]]"
          fields:
            name:
              multiline: true
            fruit:
              type: list # or `choice`
              values:
                - Apples
                - Bananas
                - Oranges
                - Peaches
```

`type: form`
`params:`
	`layout:`
	`fields:`
		`VAR_NAME:` default = text
			`multiline:` default = false
		`VAR_NAME:`
			`type:` enum : `choice`, `list`
			`values:`
				... list formats


### NO SUCH List Extension Type

- Even if we can see the `list` type inside `form` field available types
- Extension type `list` can't be used as a `vars` variable
- You can only use the `list` type inside of the `form` extension type 
-> `list` is a `choice` extension type alias itself available only with the `form` shorthand for `replace` . Or in other words, it is just a `form` GUI variation for the `choice` extension type.  

## Focus on the Form Extension

Here using a **form extension with the main replace statement**
```yaml
  - trigger: ":form"
    replace: "Hey {{form1.name}}, how are you ?" 
    # here form1.name because the variable name comes from the variable form1, 
    # so we need to retrieve it through the form1.name statement.
    vars:
      - name: "form1"
        type: form
        params:
          layout: "Hey [[name]], how are you?"
```


Here using the **main replace statement form extension shorthand**
```yaml
  - trigger: ":form"
    form: "Hey [[name]], how are you?"
```


```yaml
  - trigger: ":greet"
    form: |
      Hey [[name]],
      Happy Birthday!
```

```yaml
  - trigger: ":greet"
    form: |
      Hey [[name]],
      [[text]]
      Happy Birthday!
    form_fields:
      text:
        multiline: true
```

```yaml
  - trigger: ":form"
    form: |
      [[choices]]
    form_fields:
      choices:
        type: choice
        values:
          - First choice
          - Second choice
```

```yaml
  - trigger: ":form"
    form: |
      [[choices]]
    form_fields:
      choices:
        type: choice
        values: |
          First choice
          Second choice
```

```yaml
  - trigger: ":form"
    form: |
      [[choices]]
    form_fields:
      choices:
        type: list
        values:
          - First choice
          - Second choice
```



## Extension Attributes

### depends_on attribute
```yaml
global_vars:
  - name: one
    type: shell
    params:
      cmd: "echo one"
  - name: two
    type: shell
    depends_on: ["one"]
    params:
      cmd: "echo two"

matches:
  - trigger: ":hello"
    replace: "hello {{one}} {{two}}"
```

```yaml
    depends_on: ["one"]
```


### inject_vars attribute

`inject_vars: false`

```yaml
  - trigger: ":hello"
    replace: "hello {{output}}"
    vars:
      - name: output
        type: echo
        inject_vars: false
        params: 
          echo: "{{var}}"
```



___
## WIP
___

- [ ] Find better names for Extensions categories OR find a more "natural" order for them
- [ ] Be more explicit on the details already given
- [ ] Add explanations 
- [ ] Add more examples to show what's possible or not


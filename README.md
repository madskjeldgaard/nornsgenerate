# nornsgenerate

A command line tool for generating a basic project for the monome [norns](https://monome.org/docs/norns/) sound computing platform. 

The tool generates a basic main lua file and optionally a custom SuperCollider engine as well.

## Usage:
If you just supply a projectname, it will only generate a basic lua file project, but if you add a second argument for the engine name it will 

```bash
nornsgenerate <projectname> <enginename> <target_dir>
```
If no enginename is supplied, it will not generate an engine file
If no target dir is supplied it will use the result of `cwd` (the directory from which you executed the command)

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

The default engine it generates contains a very simple patch which takes a stereo input and adds volume control to it and panning and then spits it out again. This is to make it as easy as possible to understand the ins and outs of the SuperCollider code necessary for a custom norns engine.

### Example
```bash
nornsgenerate howdy TurboMegaOverdrive /home/mads/
```
will generate a project called `howdy` in a folder of the same name in `/home/mads`.

The project structure will look like this:

```
howdy.lua
lib/Engine_TurboMegaOverdrive.sc
```

After rebooting the norns, this should result in a custom engine now available called TurboMegaOverdrive.


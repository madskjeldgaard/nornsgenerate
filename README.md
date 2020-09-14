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

howdy.lua
```
howdy.lua
lib/Engine_TurboMegaOverdrive.sc
```

After rebooting the norns, this should result in a custom engine now available called TurboMegaOverdrive.

```lua
-- scriptname: howdy
-- v1.0.0 @mads

local viewport = { width = 128, height = 64, frame = 0 }

-- Parameters for custom engine
engine.name = "Yo"

-- Amplitude
params:add_control("amp", "amp", controlspec.new(0.0, 1.0, 'lin', 0, 0.5, 'amps'))
params:set_action("amp", function(x) engine.amp(x) end)

-- Panning
params:add_control("pan", "pan", controlspec.new(-1.0, 1.0, 'lin', 0, 0.5, 'pans'))
params:set_action("pan", function(x) engine.pan(x) end)

-- initialization
function init()
	engine.load("Yo")
end


-- key actions: n = number, z = state
function key(n,z)
end


-- encoder actions: n = number, d = delta
function enc(n,d)
	if n == 2 then
		params:delta("amp", d)
	elseif n == 3 then
		params:delta("pan", d)
	end
end

function redraw()
	screen.clear()
	screen.move(viewport.width/2,viewport.height/2)
	screen.text("this is howdy")
	screen.update()
end

-- deinitialization
function cleanup()
end
```

lib/Engine\_TurboMegaOverdrive.sc:
```supercollider
Engine_TurboMegaOverdrive : CroneEngine {
	var synth;

	*new { arg context, doneCallback;
		^super.new(context, doneCallback);
	}

	alloc {
		SynthDef.new(\boringvolume, {|inL=0, inR=1, out=0, amp=0.5, pan=0|
			var sig = In.ar([inL, inR]);
			sig = sig * amp;
			sig = Balance2.ar(sig[0], sig[1], pan);
			Out.ar(out, sig)
		}).add;

		// Sync with audio norns' sc server
		context.server.sync;

		/*
		The audio context's fx group:
		context.xg

		The in bus:
		context.in_b

		The out bus:
		context.out_b
		*/

		synth = Synth.new(\boringvolume, 
			[
				\inL, context.in_b[0].index,
				\inR, context.in_b[1].index,
				\out, context.out_b.index 
			],
			target: context.xg // The fx group of Crone
		);

		// Expose commands to lua like this:
		this.addCommand("amp", "f", {|msg|
			var val = msg[1];
			synth.set(\amp, val)
		});

		this.addCommand("pan", "f", {|msg|
			var val = msg[1];
			synth.set(\pan, val)
		});

	}

	free {
		synth.free;
	}
}
```

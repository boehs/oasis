- Components
	- `<Raw/>` [[escaped-and-raw-components]]
	- `<Escape/>` [[escaped-and-raw-components]]
- Compiler
	- AllowedCustomElements [[custom-elements]]
- API
	- `Astro.defer(execute: function,point: number = 1)`
		- Runs after rest of site is built
			- Site.0
			- Astro.defer.1
			- Astro.defer.2
			- Astro.defer.2
			- Astro.defer.3
			- etc
	- `Astro.compile(thing: function)` run at compile time, entire `Astro.compile(...)` is replaced with whatever `thing` returns (`let n = Astro.compile(function() {return 5 + 5})` is compiled into `let n = 10` once built)
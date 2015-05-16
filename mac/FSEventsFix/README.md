# FSEventsFix

Works around a long-standing bug in realpath() that prevents FSEvents API from monitoring certain folders on a wide range of OS X releases (10.6-10.10 at least).


## The Bug

The underlying issue is that for some folders, realpath() call starts returning a path with incorrect casing (e.g. "/users/smt" instead of "/Users/smt"). FSEvents is case-sensitive and calls realpath() on the paths you pass in, so an incorrect value returned by realpath() prevents FSEvents from seeing any change events.

See the discussion at [thibaudgg/rb-fsevent#10](https://github.com/thibaudgg/rb-fsevent/issues/10) about the history of this bug and how this library came to exist.


## Usage

Add `FSEventsFix.h` and `FSEventsFix.c` to your project and call `FSEventsFixInstall()`.

It is recommended that you install FSEventsFix on demand, using `FSEventsFixIsBroken` to check if the folder you're about to pass to `FSEventStreamCreate` needs the fix. Note that the fix must be applied before calling `FSEventStreamCreate`.

`FSEventsFixIsBroken` requires a path that uses the correct case for all folder names, i.e. a path provided by the system APIs or constructed from folder names provided by the directory enumeration APIs.

You can build with `FSEVENTSFIX_DUMP_CALLS` preprocessor macro set to `1` to have the library print the installation status and log all calls to realpath() to stderr.

Build with `FSEVENTSFIX_RETURN_UPPERCASE_RESULT_FOR_TESTING` macro set to `1` to make `realpath()` always return uppercase strings; it's a great way to check that the library works.

You can check the installation result by reading FSEventsFix environment variable. Possible values (defined as preprocessor macros in `FSEvents.h`) are:

- (not set or empty string): not yet installed

- `installed`: successfully installed

- `failed`: installation or self-test failed

- `unnecessary`: the current version of OS X doesn't exhibit the bug (reserved for when Apple finally fixes the bug; not currently used)

- `disabled`: not used by the library, but if you set the variable to this value, the library will not be installed

Please don't set FSEventsFix to any other values.


## Implementation

FSEventsFix uses Facebook's fishhook to replace the system `realpath()` function with a custom implementation that does not screw up directory names; FSEvents will then invoke our custom implementation and work correctly.

Our implementation of realpath is based on the open-source implementation from OS X 10.10, with a single change applied (enclosed in `BEGIN WORKAROUND FOR OS X BUG` ... `END WORKAROUND FOR OS X BUG`).


## Acknowledgments

Thanks go to:

* Travis Tilley, for spending many hours researching this bug with me

* Bryan Jones, for finally getting Apple to reveal a critical piece of information (that FSEvents calls realpath) that sparkled the idea for this approach, and for testing its feasibility, and generally for persisting in trying to resolve it and getting our lazy asses to move

* countless users of LiveReload, CodeKit and Guard who suffered this bug for years, sent us bug reports, tested builds and allowed us to poke in their machines


## License

See [FSEventsFix.c](FSEventsFix.c) file for license & copyrights, but basically this library is available under a mix of MIT and BSD licenses.


## Version History

0.9.0 (May 16, 2015) - initial beta release with a version number.
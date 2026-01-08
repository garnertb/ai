
 * Include a usage, description and optional examples with this format:

  ```bash
  #!/bin/bash
  #/ Usage: ghe-this-is-my-script [options] <required_arg>
  #/
  #/ This is a brief description of the script\'s purpose.
  #/
  #/ OPTIONS:
  #/   -h | --help                      Show this message.
  #/   -l | --longopt <required_arg>    An option.
  #/   -c <required_arg>                Another option.
  #/
  #/ EXAMPLES: (optional section but nice to have when not trivial)
  #/
  #/    This will do foo and bar:
  #/      $ ghe-this-is-my-script --longopt foobar -c 2
  #/
  set -e
  ```

If there are no options or required arguments, the `OPTIONS` section can be ignored.

* scripts must accept both -h and --help arguments and print the usage information and exit 2.

  ```bash
  #!/bin/bash
  #/ Usage: ghe-this-is-my-script [options] <required_arg>
  #/
  #/ This is a brief description of the script's purpose.
  set -e

  if [ "$1" = "--help" ] || [ "$1" = "-h" ]; then
    grep '^#/' <"$0" | cut -c 4-
    exit 2
  fi

  ```
* avoid bash arrays, they are not portable
* Use `test` or `[` whenever possible

```bash
test -f /etc/passwd
test -f /etc/passwd -a -f /etc/group
if [ "string" = "string" ]; then
  true
fi
```

* Scripts may use `[[` for advanced bash features

```bash
if [[ "$(hostname)" = *.iad.github.net ]]; then
  true
fi
```

* Scripts may use Bash `for` loops

Preferred:

```bash
for i in $(seq 0 9); do
done
```

or:

```bash
for ((n=0; n<10; n++)); do
done
```

* Use `$[x+y*z]` for mathematical expressions

```bash
local n=1
let n++
n=$[n+1] # preferred
n=$[$n+1]
n=$((n+1))
n=$(($n+1))
```

* *Use variables sparingly

Short paths and other constants should be repeated liberally throughout code since they
can be search/replaced easily if they ever change.

```bash
DATA_DB_PATH=/data/user/db
mkdir -p $DATA_DB_PATH
rsync $DATA_DB_PATH remote:$DATA_DB_PATH
```

versus the much more readable:

```bash
mkdir -p /data/user/db
rsync /data/user/db remote:/data/user/db
```

* Use lowercase variables for locals and internal veriables, and uppercase for variables inherited or exported via the environment

```bash
#!/bin/bash
#/ Usage: [DEBUG=0] process_repo <nwo>
nwo=$1
[ -n $DEBUG ] && echo "** processing $nwo" >&2

export GIT_DIR=/data/repos/$nwo.git
git rev-list
```

## Use `${var}` for interpolation only when required

```bash
greeting=hello
echo $greeting
echo ${greeting}world
```

* Use functions sparingly, opting for small/simple/sequential scripts instead whenever possible
* Use `<<heredocs` when dealing with multi-line strings
  - `<<eof` and `<< eof` will allow interpolation
  - `<<"eof"` and `<<'eof'` will disallow interpolation
  - `<<-eof` and `<<-"eof"` will strip off leading tabs first
* Let Bash choose the file descriptor for you
* File descriptors other than `1` (stdio) and `2` (stderr) are free for engineers to use, but they come with a risk.
* File descriptors are shared by subprocesses within bash. If two bash scripts use the same file descriptor, `3`
 for example, then they must agree on how `3` should be used.
* Defining functions with arguments
  ```bash
  regular_func() {
    declare arg1="$1" arg2="$2" arg3="$3"
    # ...
  }
  ```
* for variadic arguments:
```bash
variadic_func() {
	local arg1="$1"; shift
	local arg2="$1"; shift
	local rest="$@"

	# ...
}
```

* Conditionals: Testing for exit code vs output

```bash
# Test for exit code (-q mutes output)
if grep -q 'foo' somefile; then
  ...
fi

# Test for output (-m1 limits to one result)
if [[ "$(grep -m1 'foo' somefile)" ]]; then
  ...
fi
```

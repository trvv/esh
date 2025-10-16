                .x+=:.
                z`    ^%    .uef^"
                    .   <k :d88E
        .u       .@8Ned8" `888E
     ud8888.   .@^%8888"   888E .z8k
    :888'8888. x88:  `)8b.  888E~?888L                (ɔ) 2025 trvv.me
    d888 '88%" 8888N=*8888  888E  888E
    8888.+"     %8"    R88  888E  888E                  ''yOUR eSHCAPE,,
    8888L        @8Wou 9%   888E  888E
    '8888c. .+ .888888P`    888E  888E
      "88888%   `   ^"F     m888N= 888>
         "YP'                 `Y"   888
                                     J88"
                                     @%
                                   :"

# About
*Esh* is a toolkit for writing powerful, portable shellscripts.

# Library

## defined
`defined` `&variable`

<details><summary>Source (starting @ line 24)</summary>

```sh
defined() {
    case $# in
        (1) :;;
        (0) throw "defined: expected name";;
        (*) throw "defined: too many arguments";;
    esac || return
    is name "$1" || throw "defined: $1: bad name" || return
    eval "case \${$1+.} in ('') return 1;; esac"
}

```

</details>

## document
`document` `?file`

<details><summary>Source (starting @ line 36)</summary>

```sh
document() {
    [ $# -le 1 ] || throw "document: too many arguments" || return
    push line linenum=0 state=docs
    while IFS= read -r line; do
        case $line in
            ('#!'*) :;;
            ('#'*)
                [ "$state" = code ] && printf '```\n\n\074/details\076\n\n'
                state=docs
                case $line in
                    ('# '*) printf '%s\n' "${line#??}";;
                    (*) printf '%s\n' "${line#?}";;
                esac
            ;;
            ('') printf '\n';;
            (*)
                [ "$state" = docs ] && printf '\n\074details\076\074summary\076Source (starting @ line %d)\074/summary\076\n\n```sh\n' "$linenum"
                state=code
                printf '%s\n' "$line"
            ;;
        esac
        linenum="$((linenum + 1))"
    done <<-EOF
$(cat ${1+"$1"})
EOF
    [ "$state" = code ] && printf '```\n\n\074/details\076\074/summary\076\n\n'
    pops line linenum state
}

```

</details>

## is
`is` `predicate` `argument`

<details><summary>Source (starting @ line 67)</summary>

```sh
is() {
    case $# in
        (2) :;;
        (0) throw "is: expected predicate and argument";;
        (1) throw "is: expected argument";;
        (*) throw "is: too many arguments";;
    esac || return
    case $1 in
        (name)          ! match '[!A-Za-z_]*|*[!A-Za-z_0-9]*' "$2";;
        (natural)       ! match '*[!0-9]*' "$2";;
		(command)       >/dev/null command -V "$2";;
        (block)         test -b "$2";;
        (stream)        test -c "$2";;
        (directory)     test -d "$2";;
        (empty)         test -s "$2";;
        (extant)        test -e "$2";;
        (file)          test -f "$2";;
        (sgid)          test -g "$2";;
        (symlink)       test -h "$2";;
        (fifo)          test -p "$2";;
        (readable)      test -r "$2";;
        (socket)        test -S "$2";;
        (terminal)      test -t "$2";;
        (suid)          test -u "$2";;
        (writeable)     test -w "$2" || { test -d "$2" && test -x "$2"; };;
        (executable)    test -x "$2";;
        (null)          test -z "$2";;
		(*)             throw "is: $2: bad predicate";;
    esac
}

```

</details>

## match
`match` `!pattern` `string`

<details><summary>Source (starting @ line 100)</summary>

```sh
match() {
    case $# in
        (2) :;;
        (0) throw "match: expected pattern";;
        (1) throw "match: expected expression";;
        (*) throw "match: too many arguments"
    esac || return
    eval "case \$2 in ($1) :;; (*) return 1;; esac"
}

```

</details>

## move
`move` `&target` `&source`

<details><summary>Source (starting @ line 112)</summary>

```sh
move() {
    test $# = 2 || throw "move: expected target and source" || return
    is name "$1" || throw "move: $1: bad name (target)" || return
    is name "$2" || throw "move: $2: bad name (source)" || return
    eval "$1=\$$2"
}

```

</details>

## out
`out` `@strings`

<details><summary>Source (starting @ line 121)</summary>

```sh
out() {
    printf '%s\n' "$@"
}

```

</details>

## peek
`peek` `&variable`

<details><summary>Source (starting @ line 127)</summary>

```sh
peek() {
    case $# in
        (1) :;;
        (0) throw "peek: expected name";;
        (*) throw "peek: too many arguments";;
    esac || return
    is name "$1" || throw "peek: $1: bad name" || return
    eval "out \"\$$1\""
}

```

</details>

## pop
`pop` `@&variable`

<details><summary>Source (starting @ line 139)</summary>

```sh
pop() {
    test $# != 0 || throw "pop: expected name(s)" || return
    while test $# != 0; do
        is name "$1" || throw "pop: $1: bad name" || return
        eval "
            if defined ${1}_; then
                out \$$1
                if defined ${1}_\$((\${${1}_} - 1)); then
                    move $1 ${1}_\$((\${${1}_} - 1))
                    # this unset is not necessary, but we might want it for cleanliness sake
                    #unset -v ${1}_\$((\${${1}_} - 1))
                else
                    unset -v $1
                fi
            else
                throw 'pop: nothing to pop'
                return 1
            fi
            ${1}_=\$((${1}_ - 1))
        "
        shift
    done
}

```

</details>

## pops
`pops` `@&variable`

<details><summary>Source (starting @ line 165)</summary>

```sh
pops() {
    pop "$@" >/dev/null
}

```

</details>

## push
`push` `@&variable?=value`

<details><summary>Source (starting @ line 171)</summary>

```sh
push() {
    test $# != 0 || throw "push: expected name or definition" || return
    while test $# != 0; do
        is name "${1%%=*}" || throw "push: $1: bad name" || return
        eval "
            if defined ${1%%=*}; then
                move ${1%%=*}_\${${1%%=*}_=1} ${1%%=*}
            else
                unset -v ${1%%=*}_\${${1%%=*}_=1}
            fi
            ${1%%=*}_=\$((${1%%=*}_ + 1))
        "
        put "$1"
        shift
    done
}

```

</details>

## put
`put` `@&variable?=value`

<details><summary>Source (starting @ line 190)</summary>

```sh
put() {
	test $# != 0 || throw "put: expected name or definition" || return
    while test $# != 0; do
		#note Might be a way to make this a single check on the string.
        is name "${1%%=*}" || throw "put: $1: bad name" || return
		if match '*=*' "$1"; then
			eval "${1%%=*}=\${1#*=}"
		else
			unset -v "$1"
		fi
        shift
    done
}

```

</details>

## quote
`quote` `string` `?delimeter`

<details><summary>Source (starting @ line 206)</summary>

```sh
quote() {
    case $# in
        (1|2) :;;
        (0) throw "quote: expected string";;
        (*) throw "quote: too many arguments";;
    esac || return
    if match '*'"\\'"'*' "$1"; then
        # shellcheck disable=SC2139
        set -- "$(alias _="$1"; alias _)" ${2+"$2"}
        printf '%s%b' "${1#??}" "${2-\\n}"
    else
        printf '%s%b' "'$1'" "${2-\\n}"
    fi
}

```

</details>

## setmask
`setmask` `option` `?!@command`

**DEPRECATED:** This will be removed once [mask](#mask) is stable. 

<details><summary>Source (starting @ line 225)</summary>

```sh
setmask() {
    case $# in
        (0) THIS=setmask throw "expected option character";;
        (1)
            case $1 in
                ([A-Za-z0-9]) eval "case \$- in (*$1*) :;; (*) return 1;; esac";;
                (*[!A-Za-z0-9_]*) THIS=setmask throw "$1: invalid option";;
                (*)
                    # TODO: Support checking for if the option exists at all or not
                    set -- "$1" "$(set +o)"
                    case $2 in
                        (*"-o $1"*) return 0;;
                        (*"+o $1"*) return 1;;
                        (*) THIS=setmask STATUS=2 throw "$1: option not found";;
                    esac
                    #eval "set +o | grep \"-o $1\" 2>/dev/null"
                ;;
            esac
        ;;
        (*)
            case $1 in
                ([A-Za-z0-9])
                    eval "
                        shift
                        case \$- in
                            (*$1*) \"\$@\";;
                            (*)
                                if set -$1; then
                                    \"\$@\"
                                    eval \"
                                        set +$1
                                        return \$?
                                    \"
                                else
                                    THIS=\"\${THIS-setmask}\" throw \"$1: failed to set option\"
                                fi
                            ;;
                        esac
                    "
                ;;
                (*[!A-Za-z0-9_]*) THIS=setmask throw "$1: invalid option";;
                (*)
                    # This depends on the User Portability Utilities being present,
                    # which I feel pretty comfortable in assuming for most platforms.
                    # Maybe add a check if anyone runs into any situations where this
                    # is actually a problem.
                    eval "
                        shift
                        # Get rid of this grep
                        if set +o | grep \"-o $1\" 2>/dev/null >&2; then
                            \"\$@\"
                        else
                            if set -o $1; then
                                \"\$@\"
                                eval \"
                                    set +o $1
                                    return \$?
                                \"
                            else
                                THIS=\"\${THIS-setmask}\" throw \"$1: failed to set option\"
                            fi
                        fi
                    "
                ;;
            esac
        ;;
    esac
}

```

</details>

## throw
`throw` `@strings`

<details><summary>Source (starting @ line 296)</summary>

```sh
throw() {
    eval "
        >&2 out \"\$@\"
        case \$- in
            (*i*) return $(($? ? $? : 254));;
            (*) exit $(($? ? $? : 255));;
        esac
    "
}

```

</details>

## wrap
`wrap` `@strings`

<details><summary>Source (starting @ line 308)</summary>

```sh
wrap() {
    test $# = 0 && return
	while test $# != 1; do
		quote "$1" ' '
		shift
	done
	quote "$1"
}

```

</details>

# Appendix
## To do
- [ ] Extend [push](#push) and [pop](#pop) for managing environment options
  - How would this work?
- [ ] Implement [mask](#mask)
- [ ] Add unsafe function variants for better performance (!)
  - Oftentimes a given function might perform checks that another function it calls internally will then do again
  - To reduce overhead we can instead use variants of those functions that do not perform the checks when we're absolutely certain nothing will break
- [ ] Add a githook to automatically regenerate [README.md](#) at pre-commit
- [ ] Implement bidirectional lists need [queue](queue) and [pull](pull)
  - Technically this will require introducing a secondary stack pointer that grows "down"
  - List length calculations would require a simple addition of the two pointers instead of simply referencing the current stack pointer

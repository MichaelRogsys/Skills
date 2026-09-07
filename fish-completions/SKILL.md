---
name: fish-completions
description: Create or update robust Fish shell completion files for CLI commands from supplied help output. Use when asked to add Fish completions, generate a `.fish` completion file, or improve CLI completions.
---

# Fish Completion Authoring

Create a precise Fish completion file from the CLI help specification supplied by the user.

## Requirements

1. Request the CLI help output if commands, options, argument names, or valid values are not provided. Do not invent unsupported flags, aliases, subcommands, or option-value enums.
2. Write the completion file to the requested location. If none is given, use `~/.config/fish/completions/<command>.fish`.
3. Do not modify unrelated files.
4. Use Fish's `complete` builtin. Give each completion a concise description.
5. Include global options only when documented. Global options must apply at every command depth.
6. Add hierarchical, context-sensitive command completions:
   - Root commands only at the root.
   - Subcommands only after their matching parent command path.
   - Suppress file completion for command words with `-f`.
   - Account for documented unique abbreviated command names if the CLI accepts them. Do not add speculative aliases.
7. Add command-specific options only for their respective command paths. For options requiring a value, use `-r`.
8. Offer static values with `-a` only when the help specification explicitly defines accepted values.
9. Use Fish-native argument completion where appropriate:
   - Local files or paths: `-F`.
   - Local directories: `-f -a '(__fish_complete_directories)'`.
   - Remote/service paths, IDs, names, emails, and free-form text: do not use local filesystem completion.
   - Do not try to infer positional argument roles from the current token when variable-length positional arguments make it unreliable.
10. Keep helper functions private, prefixed with `__<command>_`, and place them in the completion file. Helpers must use `commandline -opc` and ignore options when evaluating command depth or path.
11. Quote Fish command substitutions and conditions correctly. Avoid Bash syntax.

## Recommended Structure

```fish
function __example_words
    commandline -opc | string match -rv '^-' 
end

function __example_at_depth
    set -l words (__example_words)
    set -e words[1]
    test (count $words) -eq $argv[1]
end

function __example_command_is
    set -l words (__example_words)
    set -e words[1]
    test (count $words) -ge (count $argv); or return 1

    for index in (seq (count $argv))
        test "$words[$index]" = "$argv[$index]"; or return 1
    end
end

complete -c example -n '__example_at_depth 0' -f -a parent -d 'Parent command'
complete -c example -n '__example_command_is parent; and __example_at_depth 1' -f -a child -d 'Child command'
complete -c example -n '__example_command_is parent child' -l format -r -a 'json yaml' -d 'Output format'
```

Adapt helper names and conditions to the command. If abbreviated command paths are supported, make conditions recognize both the full and documented abbreviated forms.

## Validation

1. If Fish is available, run `fish -n <completion-file>`.
2. If syntax validation succeeds, run a small representative check such as:

```sh
fish -C 'source <completion-file>' -c 'complete -C "<command> <parent> "'
```

3. Report the created file and validation result concisely. If Fish is unavailable, state that syntax validation could not be run.

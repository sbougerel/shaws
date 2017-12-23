# shaws

`shaws` helps you to manage temporary shell sessions for AWS CLI when using an
MFA token.

    shaws enter TOKEN_CODE

Get a session token and launches a child shell with the proper environment
variables set. Subsequent `aws` calls will all use the temporary session token.

To exit a `shaws` session, simply type `exit`.

    shaws run TOKEN_CODE aws ec2 describe-instances

Creates a unique session for the command executed.

MFA token are configured permanently to be used for a particular profile with:

    shaws --profile PROFILE config-attach MFA_SERIAL


When `shaws` is called without argument, `shaws` tells you whether you are in a
session or not.

Within a `shaws` session, the environment variable `SHAWS_SESSION` is defined
and contains the expiry time of the session. This variable is read when calling
`shaws` without arguments. If you unset the variable within your session,
`shaws` will think you have exited your session.



## Options

    -p,--profile PROFILE
`shaws` sets `AWS_PROFILE` before calling `aws`. Thus you are expected to have
your AWS credentials already properly set up. If the argument is not given, no
`--profile` will be given to `aws` which results in using the `default` profile
credentials. See `aws` cli documentation.

    -r,--role ROLE_ARN
Calls `aws sts assume-role` instead of `aws sts get-session-token` and pass the
given `ROLE_ARN` on the command line. Only used with `enter` command.


## Commands

    config-attach MFA_SERIAL
`shaws` need to use an MFA token with nearly every call. Leveraging on `aws
configure`, `shaws` define the variable `shaws_mfa_serial` for your profile.

    config-detach
`shaws` removes the token associated with your profile.

    list-devices
Equivalent to `aws iam list-mfa-devices --user-name <your username>`. `shaws`
deduce your username from your profile and list your configured devices to make
it easy to use the `config-attach` command.

    enter TOKEN_CODE
Enter a `shaws` session. Exit by typing `exit` within the session. See also the
`-r` option.

    run TOKEN_CODE STRING ARGS...
Simlar to POSIX shells when the `-c` option is used. Commands are read from
`STRING`. If there are arguments after the string, they are assigned to the
positional parameters, starting with `$0`.

    run TOKEN_CODE - ARGS...
Simlar to POSIX shells when the `-s` option is present. Commands are read from
the standard input. If there are arguments after the string, they are assigned to the
positional parameters, starting with `$0`.

    help
Shows this readme page.


## Usage example

With the assumption that you have not used `aws` before:

    $ aws configure
    // follow the instruction to create the default profile
    $ shaws list-devices
    // Find MFA tokens associated with your profile
    $ shaws config-attach MFA_DEVICE
    $ shaws enter TOKEN_CODE
    // start using aws cli...

And next time, just:

    $ shaws enter TOKEN_CODE


## Installation

Copy `shaws` in `/usr/local/bin/shaws` and make it executable with `chmod a+x /usr/local/bin/shaws`.

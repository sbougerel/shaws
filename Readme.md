# shaws

`shaws` manages AWS CLI Multi Factor Authentication for you. It
creates a new shell with the session tokens stored in environment
variables so all 'aws' calls within the new shell use this session
token.

Since `shaws` uses a new shell for each token requested, you can use
multiple sessions in parallel in different shells without conflicts,
or simply copy the environment of one session into another to use the
same token.

`shaws` is natural to use and does not require any particular
configuration instead of the one already recommanded by AWS.

`shaws` can be used to simply request session tokens or to assume a role. The
action taken by `shaws` depends on your `./aws/config`, see
[pre-requisites](#Pre-requisites) and [examples](#Examples) below for more
information.

Within the new shell, the environment variable `SHAWS_SESSION` is
defined and contains the expiry time of the session. This variable is
read when calling `shaws` without arguments. If you unset the variable
within your session, `shaws` will assume you have exited your session.

`shaws` sets `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and
`AWS_SESSION_TOKEN` in the new shell. These variables have precedence
over any other variables for AWS CLI.


## Syntax

    shaws PROFILE COMMAND [ARGS]

If you're going to use a specific profile repeatidly, aliasing comes
in handy, e.g. with a `default` profile:

    alias shawsd='shaws default'


## Commands

To see the current session's settings:

    shaws


To simply create a new session with 'shaws' using 2FA:

    shaws PROFILE enter TOKEN_CODE


Alternatively, to execute only a single command:

    shaws PROFILE run TOKEN_CODE aws ec2 describe-instances


MFA token are configured permanently to be used for a particular profile with:

    shaws PROFILE attach MFA_SERIAL


When `shaws` is called without argument, `shaws` tells you whether you are in a
shell with and active session token or not.


For more details:

    shaws help


Within the new shell, the environment variable `SHAWS_SESSION` is defined and
contains the expiry time of the session. This variable is read when calling
`shaws` without arguments. If you unset the variable within your session,
`shaws` will assume you have exited your session.

`shaws` will set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and
`AWS_SESSION_TOKEN` in the new shell. These variables have precedence over any
other variables for AWS CLI.


## Requirements

`shaws` works on MacOS and Linux. It depends on `bash`, `binutils`, AWS CLI and
`jq` being present on the machine.


## Pre-requisites

`shaws` relies on AWS CLI internally, so it expects you to follow the
AWS CLI configuration to get session token or assume roles:
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html


## Example

Assuming you have not used AWS CLI before:

    $ aws configure
    // follow the instruction to create the default profile
    $ shaws default ls-devices
    // Find MFA tokens associated with your profile
    $ shaws default attach MFA_DEVICE
    // This will attach the device to the default profile or:
    $ shaws default attach some-profile MFA_DEVICE
    // This will attach the device to the profile some-profile


In this example, we create a session with an MFA token using the default
profile:

    $ shaws default enter TOKEN_CODE
    // Use AWS CLI without specifying profile or credentials after


In case you want to start assuming a role, first add the new role to your AWS
CLI configuration and bind it to the profile's creditials with `source_profile`;
if you do not know, you probably need `default`. You need to specify the MFA
device to use for that role in `mfa_serial`; if unsure, copy the value from your
source profile. Example `~/.aws/config`:

    [default]
    region = some-region-1
    mfa_serial = arn:aws:iam::123456789012:mfa/userid
    
    [profile assumed-role]
    role_arn = arn:aws:iam::123456789012:role/assumed-role
    source_profile = default
    mfa_serial = arn:aws:iam::123456789012:mfa/userid

Afterwards, you can create sessions for the new role with:

    $ shaws assumed-role enter TOKEN_CODE
    // Use AWS CLI without specifying profile or credentials after


## Installation

Copy `shaws` in `/usr/local/bin/shaws` and make it executable with `chmod a+x
/usr/local/bin/shaws`.

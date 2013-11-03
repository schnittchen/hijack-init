# hijack-init

Testing the userland launch of a containerized system, by starting hijack-init as the first process

__CAUTION__ Don't execute hijack-init on your system, it may render it unbootable!

## Why?

For testing system provisioning or installation.

## Background

In a container, just like on a linux system operating directly under the kernel, there is a first process whose PID is 1. This process can exec and yield a process which still has PID 1.

With lxc, we can obtain the output and the exit status of this process. We want to use these for obtaining result and diagnostics from the test.

## Goals

* we need to start `/sbin/init` at some point. It is the system under test and treated as a black box.
* we need to execute a test script, its output goes to stdout/stderr of the PID 1 process.
* we need the PID 1 process to exit with status 0 or nonzero, depending on the result of the test script.

## Different init systems

Currently, SysV Init and upstart are supported.

The main difficulty is that `/sbin/init` behaves like `telinit` if it is not the process with PID 1.

* SysV Init honors a `--init` flag that makes it act like that process even if it isn't.
* For upstart, there is no such flag, so we need it to be PID 1. We use the hack described below to make it exit with the desired exit code.

### Hacking upstart

Upstart restarts itself upon receiving a TERM signal, and we replace `/sbin/init` before that happens.
The replacement restores the original `/sbin/init` and then exits with the appropriate exit code.

It turns out that this hack is not only useful for testing the installation of an upstart system,
it is even necessary for enabling automated setup, if that setup involves starting system services.
Unlike the SysV scripts, the `start`, `stop` etc. tools require a connection to a running upstart.

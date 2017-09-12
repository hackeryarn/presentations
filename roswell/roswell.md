class: center, middle

# Roswell

---

# Setup

Mac OS X:
```bash
$ brew install roswell
```

Arch Linux:
```bash
$ yaort -S roswell
```

Linxbrew:
```bash
$ brew install roswell
```

Windows:
- [Roswell-x86_64.zip](https://ci.appveyor.com/api/projects/snmsts/roswell-en89n/artifacts/Roswell-x86_64.zip?branch=master&job=Environment%3A%20MSYS2_ARCH%3Dx86_64,%20MSYS2_BITS%3D64,%20MSYSTEM%3DMINGW64,%20METHOD%3Dcross)
- [Roswell-i686.zip](https://ci.appveyor.com/api/projects/snmsts/roswell-en89n/artifacts/Roswell-i686.zip?branch=master&job=Environment%3A%20MSYS2_ARCH%3Di686,%20MSYS2_BITS%3D32,%20MSYSTEM%3DMINGW32,%20METHOD%3Dcross)

---

# Emacs Setup

```bash
$ ros install slime
```


Add to your emacs config:
```elisp
(load (expand-file-name "~/.roswell/helper.el"))
(setq inferior-lisp-program "ros -Q run")

;; or

(load (expand-file-name "~/.roswell/helper.el"))
(setf slime-lisp-implementations
      `((sbcl    ("sbcl" "--dynamic-space-size" "2000"))
        (roswell ("ros" "-Q" "run"))))
(setf slime-default-lisp 'roswell)
```

---

# Features

- **Implementation Manager**
- Bash scripting environment
- Build utility
- Integration with CI

---

# Supported Implementations

- sbcl
- ccl
- ecl
- clisp
- abcl
- cmucl
- alisp

---

# How to manage

```bash
# default sbcl
$ ros install sbcl-bin      

# The newest released version of sbcl
$ ros install sbcl          

# default prebuilt binary of ccl
$ ros install ccl-bin       

# A specific version of sbcl
$ ros install sbcl/1.2.0    

# Listing the installed implementations
$ ros list installed

```

---

# Features

- Implementation Manager
- **Bash scripting environment**
- Integration with CI

---

# Starting a project

```bash
$ ros init fact
```

Generates fact.ros with:

```lisp
#!/bin/sh
#|-*- mode:lisp -*-|#
#|
exec ros -Q -- $0 "$@"
|#
(defun main (&rest argv)
  (declare (ignorable argv)))
```

---

# A little bit of code

```lisp
#!/bin/sh
#|-*- mode:lisp -*-|#
#|
exec ros -Q -- $0 "$@"
|#

(defun fact (n)
  (if (zerop n)
      1
      (* n (fact (1- n)))))

(defun main (n &rest argv)
  (declare (ignore argv))
  (format t "~&Factorial ~D = ~D~%" n (fact (parse-integer n))))
```

---

# Running the program

```bash
$ fact.ros 3
Factorial 3 = 6

$ fact.ros 10
Factorial 10 = 3628800
```

---

# Distributing scripts on Quicklisp

Add all `.ros` files to `roswell` folder at root of your project.

---

# Install scrips

```bash
$ ros install <library name>
```

```bash
$ ros install clack

# run our server
$ clackup
```

*rquires `~/.roswell/bin` to be in PATH*

---

# Features

- Implementation Manager
- Bash scripting environment
- **Build utility**

---

# Supported test environments

- Travis CI
  - No official common lisp support
  - Roswell solves this

- Circle CI
  - Newer packages are supported
  - Runs faster

- Coveralls
  - provides code coverage results
  - runs with cl-coveralls

---

layout: true
.left-column[
#### Travis CI
]

.right-column[
{{content}}
]

---

In your `.travis.yml`:

```yaml
language: common-lisp
sudo: required

install:
  - curl -L https://raw.githubusercontent.com/roswell/roswell/release/scripts/install-for-ci.sh | sh

# additional settings go here

script:
  - ros -s prove -e '(or (prove:run :quri-test) (uiop:quit -1))'
```

---

Test with multiple lisps:

```yaml
env:
  matrix:
    - LISP=sbcl-bin
    - LISP=ccl-bin
    - LISP=abcl
    - LISP=clisp
    - LISP=ecl
```

---

Ignoring some failures:

```yaml
matrix:
  allow_failures:
    - env: LISP=clisp
```

---

Install the latest dependencies:

```yaml
install:
  - curl -L https://raw.githubusercontent.com/roswell/roswell/release/scripts/install-for-ci.sh | sh
  # Using the latest fast-http and quri
  - ros install fukamachi/fast-http
  - ros install fukamachi/quri
```

---

Cache directories:

```yaml
cache:
  directories:
    - $HOME/.roswell
    - $HOME/.confi/common-lisp
```

---

layout: true
.left-column[
##### Travis CI
#### Circle CI
]

.right-column[
{{content}}
]

---

Basic environment:

```yaml
machine:
  environment:
    PATH: ~/.roswell/bin:$PATH
    LLVM_CONFIG: /usr/lib/llvm-3.6/bin/llvm-config
```

---

Pre dependencies:

```yaml
dependencies:
  pre:
    - sudo bash -c 'echo "deb http://mirrors.kernel.org/ubuntu vivid main universe" >> /etc/apt/sources.list'
    - sudo apt-get update
    - apt-cache search pocl 
    - apt-cache search icd
    - apt-cache search opencl
    - sudo apt-get install -y libltdl3-dev libhwloc-dev ocl-icd-opencl-dev g++-4.8 clang-3.6 libclang-3.6-dev llvm-3.6-dev libedit-dev
    - ./build-pocl.sh
    - sudo make -C pocl install
    - curl -L https://raw.githubusercontent.com/roswell/roswell/master/scripts/install-for-ci.sh | sh
    - ros install ccl-bin
    - git clone https://github.com/cffi/cffi.git ~/lisp/cffi
```

---

Post dependencies:

```yaml
  post:
    - ros install cffi
    - ros install cffi-grovel
    - ros install eazy-opencl
```
---

Cache directories:

```yaml
  cache_directories:
    - ~/.roswell/
    - pocl
```

---

Run tests:

```yaml
test:
  override:
    - ros -L sbcl-bin testscr.ros
    - ros -L ccl-bin testscr.ros
```

---

layout: true
.left-column[
##### Travis CI
##### Circle CI
#### Coveralls
]

.right-column[
{{content}}
]

---

Using `run-prove`:

```yaml
language: common-lisp
sudo: required

env:
  global:
    - PATH=~/.roswell/bin:$PATH
    - COVERAGE_EXCLUDE=t/
  matrix:
    - LISP=sbcl-bin COVERALLS=true
    - LISP=ccl-bin
    - LISP=abcl
    - LISP=clisp
    - LISP=ecl

matrix:
  allow_failures:
    - env: LISP=clisp

install:
  - curl -L https://raw.githubusercontent.com/roswell/roswell/release/scripts/install-for-ci.sh | sh

script:
  - run-prove quri-test.asd
```

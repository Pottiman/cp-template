# Competitive Programming Template

This is a mostly platform independent template, most scripts are written for CMake, which is required as build system.

## Usage

1. Change project name in root `CMakeLists.txt`
2. Run CMake, e.g. via `cmake -B cmake-build-debug .` (automatically done by CLion)
3. Run `contests/add_contest NAME`
4. Then, either
    * Download `samples-TASK.zip` files to `contests/NAME` and run `contest/NAME/load_tasks`, or
    * Run `contest/NAME/add_task TASK`
5. Write code (this is the important part)
6. Run `ctest` in the tasks cmake binary directory (`cmake-build-debug/contests/NAME/TASK` if configured according to step 2) to test it

## Features

### Contest and task management

* `contests/add_contest NAME` to create a new contest `NAME`.
  > This invokes `scripts/add_contest.cmake` to create a contest folder, using `templates/contest.cmake` and `templates/contest/*`.
* `contests/NAME/add_task TASK` to create a new task `TASK` in contest `NAME`.
  > This invokes `scripts/add_task.cmake` to create a task folder, using `templates/task.cmake`, `templates/template.cpp` and `templates/task/*`.
* `contests/NAME/load_tasks` creates a task for each `samples-TASK.zip` in contest `NAME` and adds the samples contained in the zip file.
  > This invokes `scripts/load_tasks.cmake`, which uses `scripts/add_task.cmake` to create task folders.
* `contests/NAME/TASK/add_sample NAME` creates a sample for the given task (both `NAME.in` and `NAME.out`).
  > This is just a bash script, but rather simple, so it should be easily portable.

### Automatic testing of all samples

Run `ctest` in the cmake build directory corresponding to a task (in CLion: `cmake-build-TYPE/contests/NAME/TASK`) to run all samples. Add `--output-on-failure` for more detail (e.g. solution diff). Add `-j 8` and/or `--progress` if you feel like it.

Each time `cmake` is run (the project is reloaded), `ctest` tests are generated.
Each task receives a build test (as testing is performed via a script, the test runner does not have to be built).
For each `SAMPLE` of the task, a test is created which runs the task executable with `SAMPLE.in` as input and compares the output with `SAMPLE.out`.
The test fails if:

* the execution does not finish within 5 seconds (configurable in `config.cmake`)
* the executable exits with a non-zero exit code (usually a run error, error output is printed to console if using `--output-on-failure`)
* the output does not match the desired output (wrong answer, diff output is printed to console if using `--output-on-failure`)

Program output is saved to `SAMPLE.result`, diff output (if any) is saved to `SAMPLE.result.diff`, error output (if any) is saved to `SAMPLE.result.err`.
The sample tests are skipped if the build fails.

> There are two test runner scripts, `perform_test.sh` for UNIX and `perform_test.cmake` for other platforms. `perform_test.sh` terminates itself with `SIGSEGV` to make `ctest` output `Exception` instead of `Failed` to allow for a quick distinction between run errors and wrong answers. `perform_test.cmake` does not have this capability, so both run errors and wrong answers are reported as `Failed`.
>
> `perform_test.cmake` uses `diff` to compare outputs, this might need to be changed based on the setup.
>
> `perform_test.sh` uses `diff`, `head` and `wc` (although the latter two are not strictly required).

## Installing and Updating

The simplest way to install this is to clone this repository.
Then you can add this repository as upstream remote (`git remote add upstream REPO_URL`) and change the `origin` to your repository.

If `upstream` is set up, you can `git pull upstream master` to update to the latest version.
The template is structured such that, if at all possible, new features also apply to existing tasks.

If you already have an existing repository, you can add this repository to its history:
1. (optional) rename/move files you know will conflict
2. `git remote add upstream REPO_URL`.
3. `git pull --allow-unrelated-histories upstream master`.
   This is most certain to result in conflicts, especially in `CMakeLists.txt`.
   Mostly you can just pick the remote files in case of conflict, unless you know that you do not.
   If you feel daring, you can specify `-s recursive -X theirs` to automatically pick remote files during merge.
4. Incorporate your existing files by creating the appropriate contests and tasks and copying the respective source files.



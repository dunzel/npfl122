# Skeleton of C++ AZQuiz MCTS Implementation

This directory contains a skeleton if C++ implementation of MCTS
and self-play game simulation for AZQuiz assignments.

## Prerequisities

You need to
```
pip3 install [--user] pybind11
```
in order to be able to compile the project.

## Compilation

To compile the project, you can run for example
```
python3 setup.py build --build-platlib .
```
which creates the binary `az_quiz_cpp` module in the current directory.

## ReCodEx

The C++ implementation can be used in ReCodEx -- just submit also all the
C++ headers and sources, plus the `setup.py` module. When `setup.py` module
is submitted to ReCodEx, the above compilation command is automatically
run before importing your module; any compilation error should be reported.

## Performance

The notable property of the C++ implementation is that it can run self-play
simulation in several threads in parallel, batching evaluation requests from
all the threads. This allows large speedup both in CPU-only and GPU setups,
as indicated by the below table measurning approximate running times of
generating 1000 self-play games and performing 1000 training updates with
batch size of 512.

| Implementation | Device | Workers (parallel MCTSes) | Time |
|:---------------|:-------|--------------------------:|-----:|
| Python | 1 CPU  |   – | 2359.2s |
| C++    | 1 CPU  |   1 | 1190.3s |
| C++    | 1 CPU  |   4 |  613.3s |
| C++    | 1 CPU  |  16 |  483.1s |
| C++    | 1 CPU  |  64 |  403.8s |
| C++    | 4 CPUs |   1 |  912.8s |
| C++    | 4 CPUs |   4 |  408.2s |
| C++    | 4 CPUs |  16 |  204.7s |
| C++    | 4 CPUs |  64 |  166.2s |
| C++    | GPU    |  64 |   42.4s |
| C++    | GPU    | 128 |   29.7s |
| C++    | GPU    | 256 |   24.8s |
| C++    | GPU    | 512 |   21.3s |

## API Documentation

The provided implementation uses C++-17 and contains:
- `az_quiz.h`, which is a C++ reimplementation of `az_quiz.py`;
- `az_quiz_cpp.cpp`, which is the implementaton of the Python `az_quiz_cpp` module.
  It provides three methods:
  - ```python
    mcts(
        board: np.ndarray,
        to_play: int,
        network: Callable[[np.ndarray], tuple[np.ndarray, np.ndarray]],
        num_simulations: int,
        epsilon: float,
        alpha: float,
    ) -> np.ndarray
    ```
    Run a MCTS and returns the policy computed from the visit counts of the
    root children. The `network` is a callable, which given a batch of game
    representations produces a batch of policies and batch of value functions.
  - ```python
    simulated_games_start(
        threads: int,
        num_simulations: int,
        sampling_moves: int,
        epsilon: float,
        alpha: float,
    ) -> None
    ```
    Start the given number of threads, each performing one self-play game
    simulation.
  - ```python
    simulated_game(
        network: Callable[[np.ndarray], tuple[np.ndarray, np.ndarray]],
    ) -> list[tuple[np.ndarray, np.ndarray, float]]
    ```
    Given a callable `network`, run a parallel MCTS self-play simulations (using
    threads created by the `simulated_games_start` call). Once the first game
    finishes, it is returned as a list of triples _(game representation, policy,
    value function)_.

The implementation contains all Python ↔ C++ conversions and thread synchronization.
You need to implement:
- `AZQuiz::representation` to suitably represent the given game;
- `::mcts` implementing the MCTS;
- `::worker_thread` generating a simulation of a self-play game.

Note that all other functionality is assumed to be provided by the Python
implementation (network construction, GPU utilization, training cycle, evaluation, …).

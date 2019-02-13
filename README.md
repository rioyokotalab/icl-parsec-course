# icl-parsec-course

Short course for teaching ICL Parsec's PTG DSL.

# Documentation

If you're stuck, refer to the [parsec tutorials](http://icl.utk.edu/parsec/news/news.html?id=378) for more info.

Also have a look at this [wiki](https://bitbucket.org/icldistcomp/parsec/wiki/Home).

This [page](https://bitbucket.org/icldistcomp/parsec/wiki/writejdf) in the wiki is particularly useful
for understanding the flow syntax.

# Install

Clone the parsec sources: `git clone https://bitbucket.org/icldistcomp/parsec.git`.

`cd` into the cloned directory and then run:
```
mkdir build
cd build
ccmake ..
```
Once inside `ccmake`, configure the build such that compiling of DPLASMA is disabled.
Then generate the Makefile through `ccmake`. See this [tutorial](https://graal.ens-lyon.fr/diet/UsersManualDIET2.4/node22.html) if you're stuck.

Set the `PKG_CONFIG_PATH` variable to point to the `parsec.pc` file in the `build/`
directory. Just do a `find . -name "parsec.pc"` in the `build` directory if you cannot
find the location.

Set the `LD_LIBRARY_PATH` variable to point to folder that contains `libparsec.so` in the
`build/` directory.

# Data distribution

Data shape in parsec is defined using MPI data types. Parsec also uses a contruct called `arena`
that it uses for internally managing the data type. Arenas are more abstract data types used for
managing this data. They are mainly used for knowing how to allocate temporary memory for the data.

# Various terminologies

## Arena

An arena type is similar to MPI datatypes. It permits us to tell parsec about a 'default data type'
that is created when you use NEW in one of your tasks.

For example, if you want to say that the default data type that will be passed around by MPI
will be a single `double` number, you can use this code (assume that the JDF example is called
`matmul`):
```
parsec_arena_construct(tp->arenas[PARSEC_matmul_DEFAULT_ARENA],
    sizeof(double), PARSEC_ARENA_ALIGNMENT_SSE,
    parsec_datatype_double_t);
```

### How is `parsec_arena_construct` different from `parsec_data_create`?

### Defining custom parsec data types
In order to make parsec work with custom data types (like self-constructed matrices), it
is necessary to define your own data types that parsec can use for building custom MPI
data types which it can transmit to other tasks.

## Data flows

This is the core of the parsec system that allows you to write data flows that map the actual flow
of data in your application. Here's how you can write good data flows:

### Introduction

### READ dataflows

### RW dataflows

### WRITE dataflows

### CTL dataflows

### Defining exactly which data is supposed to flow
The `data_of` parameter inside the `parsec_data_collection_t` struct defines a function
pointer that you can assign to that will return a data pointer of type `parsec_data_t`
when you the particular task is called.

#### Question: How do you use data_of to return different parts of the array present at a node when the task parameters differ?
See the matrix creation and manipulation file. There is a sample
`data_of` function where it shown that it should recreate a `parsec_data_t`
type and return that with the required data encapsulated inside.

## Body
The body is the code that is called when the task receives the data that is dependent on.
I think the flow is `receive objects -> process using body -> send to others`.

# Usage
For an overview of parsec, first go through the slides.pdf file and see the structure
of a JDF file and other basics. Then go through each example below.

## Basic matrix multiplication
In this example we will write a simple SUMMA matrix-matrix multiplication algorithm using
parsec. The top part of the `matmul.jdf` file contains the parsec task and the `main` function
contains code for working with the parsec context.

### Tasks
A task is a basic 'unit' that computes and communicates.

### Main function
The main function should contain the following components:

* MPI initialization.
* Parsec initialization.
* Assinging data collection metadata to parsec context.
  - Rank of the process where computation should take place (`rank_of`).
  - Virtual process ID of the data possessed locally (`vpid_of`).
  - Unique key associated with the data (`data_key`).
  - Function that returns metadata on the data contained on this node (`data_of`).
* Create a taskpool.
* Submit the taskpool to the context and start the context.
* Wait for the context to finish computing.
* Free parsec data structures and stop parsec context.

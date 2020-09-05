# Relevant work
Most of the relevant work can be found under https://github.com/hasktorch/hasktorch/pulls/andredaprato. One PR is still
outstanding which contains most of the finalizing changes and documentation. 

Some benchmark data can be found on this branch
 https://github.com/andredaprato/hasktorch/tree/bench-data.  The current state
 of the benchmarks don't feel particularly convincing or meaningful so they
 aren't merged into the main branch. Haskell and python are very different
 runtimes so synthetic measurements are a bit difficult to derive much info from
 in my opinion. I think possibly a better line of benchmarks to be explored
 would be to show how easily you can write a very parallelized preprocessing
 pipeline in hasktorch compared to python libraries. Another possible benchmark 
 could be to compare real dataset performance, where we ensure the implementation
 of the datasets are basically identical in python and Haskell. 

# Extending 
- Batch handling for arbitrary tensor ADTS is still basically entirely manual, and it would nice to have some sort of generic 
implementation for turning a list of these into a batch that can be processed by a model.
- Better caching support, something inline with the style of
  [yogadl](https://yogadl.readthedocs.io/en/latest/index.html).
    Currently there is only an in-memory cache available. Haskell doesn't seem
  to have a well maintained LMDB library, so the choice of database for caching would need to be decided upon. Caching support with a proper database
  adds the ability to train in constant memory through streaming from the database, but also allows convenient random access. This would be really nice to have.
 - A more thorough comparison to dataloaders from other popular ML libraries, as discussed above. 

# Difficulties
- Resource cleanup was something that had to be considered, and factored into the design. We ended up returning continuations, this allows the dataloading function 
to run code after the user specifies what they want to do with the datastream. This was necessary so that we could clean up the threads we create for streaming data downstream after the user is done with the data.
- Originally samples would be streamed in randomly from a datastream if there was more than 1 thread streaming in samples. Obviously this is undesirable if one needs reproducability, so deterministic ordering of samples with parallelism was implemented with STM transactions, though there may still be some problems with memory leaks (this needs investigated further).
- Designing the API so that common deep learning workflows were convienient. We started out with a class which specified a stream of samples, but this inhibits things like retreiving arbitrary samples, and shuffling. The solution was to take an approach similar to pytorch, and have a indexable interface and a streaming interface.

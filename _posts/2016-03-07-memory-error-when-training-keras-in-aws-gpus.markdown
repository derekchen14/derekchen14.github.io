---
layout: post
title: Memory Error when Training in AWS GPUs
date: '2016-03-07 00:00:00'
tags:
- gpu
- memory-error
- deep-learning
- theano
---

For my Stanford [Convolutional Neural Networks course](http://cs231n.stanford.edu), I partnered with a brilliant friend of mine to analyze images from a collection of 40,000 digitized works of art by classifying them according to artist, genre, and location.  After some standard pre-processing, we employed a modified VGGNet architecture to achieve better than [state-of-the-art results](http://arxiv.org/pdf/1505.00855v1.pdf) on artist and genre classification. Along the way though, we hit a number of roadblocks, and saw `Error allocating 86118400 bytes of device memory (out of memory). Driver report 32735232 bytes free and 4294770688 bytes total.  Segmentation fault (core dumped)` more times than we would like to remember.  In getting our network to  run to properly, we encountered a number of problems and their solutions.

__Quick Fixes__
If you haven't done any kind of optimization yet:

  - Consider running your network on a [AWS 2.2xlarge GPU](feature.engineering/training-on-big-data/) or a similar offering from another cloud provider.  Not only will this provide a boost in power, your cloud instance will likely have greater memory to work with as well.
  - Reduce the size of your network by removing extra layers or limit the number of nodes per layer.  Think about other straightforward variables you can reduce, such as the number of examples to train on.  This will likely lower the final accuracy of the neural net, but if you're just trying to get something to run, limiting the network is a good place to start.
  - Close all other programs that are running on your computer that might be eating up memory.  [Chrome](http://lifehacker.com/why-chrome-uses-so-much-freaking-ram-1702537477) and its independent tab processes are often a hungry source for gobbling up CPU and RAM.  Use [Activity Monitor](https://support.apple.com/en-us/HT201464) from your Mac OSX toolbox to find out what other culprits might be at play.
  - Simplify the architecture of your overall system by commenting it out.  For example, if your final prediction uses ensembling from multiple models, temporarily use just a single model at a time.  The same idea applies for data augmentation, complex feature engineering, and using specialized hyperparameters.

__Diving Deeper__
This next level of ideas requires the user to dig into their code a bit to understand what is happening.  Simple editing of a copy/pasted example file will no longer suffice:

  - Compiled models potentially hold onto millions of weights and variables in addition their actual structure, and should not be stored unnecessarily.  As an example, assume you are training multiple networks at once during a grid search for hyperparameters.     During each training loop, you store the best model so that you can  use it during the testing phase.  A better use of memory though might be to store just the best params used in creating the model, rather than the full model itself.
  - Reduce your the batch size of each epoch.  While this may seem like a quick fix, it requires understanding the trade-offs between having different sized batches.  Additionally, unless the neural network was written in a clean style, changing this variable might involve touching multiple places in your code.
  - Load your input vectors as 'float32' rather than 'float64', which is the default of `numpy.load` using the `astype()` method.  In most cases, your network will not benefit from having the extra precision of the extra 32 bits, so changing the input type can be a great way to cut down your feature memory usage in half.
  - If your dataset is too large, you can try splitting your dataset into smaller chunks so that each individual piece can fit into memory.  This process can be performed by simply chunking up your data through numpy if you are planning to save the features as .npy files.  Alternatively, h5py has methods that allow the user to pull only certain slices of the data into memory at once.  Keras offers a nice wrapper around this functionality in their [I/O tools](https://github.com/fchollet/keras/blob/master/keras/utils/io_utils.py#L7).

__Finding Root Causes__
Finally, Linux command line tools can help pinpoint the exact moment where your network falls apart:

  - To find out how much CPU memory is available, you can use `top` or `watch free` to tail the logs. In order to get more human-readable output for top, press Shift+E.  Similarly, in order to get more human-readable output for free, use the `-k` or `-m` flag to get kilobytes and megabytes, respectively.  It can be quite beneficial to [study the guides](http://www.linuxnix.com/find-ram-size-in-linuxunix/) briefly to gain a better understanding of the output as opposed to blindly jumping in.
  - Get GPU memory information by using `nvidia-smi` or `intel_gpu_top` for Nvidia and Intel chips, respectively.  Many times you should know the maximum capacity of your graphics card, so be sure that the numbers you see line up with your understanding.  For the typical AWS GPU, this will be 4GB of video memory.  Once again, investing some time perusing the [documentation](https://developer.nvidia.com/nvidia-system-management-interface) or just some brief searching can prove to be invaluable.
  - Use `timer.sleep(x)` in order to pause processing so you can see what memory allocation is directly before and after you perform an action, such as creating a new variable.  Combined with the tools above, you should now be able to see exactly when memory usage starts to explode, and hopefully what source caused it to happen.
  - Once the issue has been identified, consider if any of the techniques mentioned earlier might ameliorate the situation.  Another idea is setting any data or weights to None after you are done using them.  For example, in the case where new chunks of data are loaded sequentially to train the network, setting `X_train = None` and `y_train = None` at the end of each loop could make a difference.  
  - Finally, although Python should take care of garbage collection pretty well, using `sys.stdout.flush()` to clear out any unused variables could make the difference.

Like any good programmer, I simply kept forging ahead until I was able to figure out something that worked.  Thus, for my purposes,  these methods were enough to get my network to run to completion, and I hope they will help you as well.  Although, if problems still remain despite having pushed this far, and you are adamant of running your network at full capacity, then it might be time to invest in a [more powerful chip](http://timdettmers.com/2014/08/14/which-gpu-for-deep-learning/) and develop your own custom set-up.

With practically unlimited access to resources, the folks at Google, Facebook, and Baidu certainly aren't facing the same type of issues, and instead are employing techniques well beyond the scope of this article.  However, using just hardware generally available to the public, even amateur researchers might be able to push these ideas further than I have.  If you are one of these people, please leave your tips and tricks in the comments below.
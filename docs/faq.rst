FAQ
=====

1. Can it use Python threads or asyncio?

Yes to both. Performance gains might be limited because of the GIL but this can be improved by using NumPy and PyTorch.

2. Can it use Python multiprocessing?

Yes.

3. Can it use CUDA?

Yes, CUDA is supported. By default, tensors passed in to the `process_torch(midi, audio)` function will be on the CPU and must be
returned on the CPU. But you can put them to the GPU and process them there.

4. Does NumPy/Torch use _real_ threads?

Yes they do and can really run in parallel.

5. What is the format for network audio?

It streams 32 bit floats. We found that the performance gains from int16 were negligible when running on localhost, in fact the overhead
in converting resulted in worse performance.

6. Does it support PyTorch JIT?

Yes, but you'll have to write the Python wrapper code. We thought about having first class support in the VST for drag-n-dropping checkpoints,
but it seems PyTorch is moving more towards `torch.compile`.

7. Does it support `torch.compile`?

Unfortunately, no. PyTorch 2.2.0 doesn't support `torch.compile` with Python 3.12 (which is used by the VST) yet. You can try, but
your code will throw an error. You can always upgrade the PyTorch version used by the VST using `python.exe -m pip` from a command
prompt window, if they release an update. We will try to keep the VST up-to-date with the latest PyTorch version and 2.4.0 will supposedly
support `torch.compile` for 3.12.

8. How do I get best audio quality?

In general, experiment with the latency (buffer size) in your DAW. Higher latencies mean larger buffers, so the amount of data moved in
one go is more, and the overheads of moving it are comparatively less. However, it means the amount of data your code has to process
in one go is also higher. So, you might have to experiment with the buffer size to get the best audio quality.

For networked audio, choosing buffer sizes that are powers of 2 seems to work best (128 - 2.9ms at 44.1kHz, 256, 512, 1024 etc).

9. How do I get the number of channels/samples in the input to my process function?

In Torch and NumPy, `num_channels, num_samples = audio.shape`. If you're using basic lists, `num_channels = len(audio)` and `num_samples = len(audio[0])`.

These values are not fixed and not known until the `process` function is called, so if you're initializing anything that depends on them, it'll have to
happen on the first call to the process function (unless you're prepared to guess).

Left and right channels always have the same number of samples.

10. Does my returned audio have to be the same shape as the input?

Yes.

11. Does my returned MIDI have to be the same shape as the input?

No, it can be an empty list (`[]`) or up to 33 messages long (one MIDI message is 3 bytes and we provided for up to 100 bytes per call to `process`. That means
up to 33 note on/note off, parameter changes etc per few milliseconds).
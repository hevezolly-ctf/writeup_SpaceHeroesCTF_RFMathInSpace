# writeup for "RF Math in Space" task in Space Heroes CTF

### description:
>### I'm sure no ones listening....right?!
>Someone thought it would be a good idea to transmit their access keys to their constellation in the clear. Luckily you have a subscription to an RF monitoring service.


So as a task we are given with two files: [samples.bin](https://drive.google.com/file/d/1-6QrcO8_wc0NJ1stArNTyhseiaWisl2X/view?usp=sharing) 
and [generate_samples.grc](generate_samples.grc). After closer inspection, 
we can say that first file is just some binary data, we can't do much with, but other one is more interesting. 
After openning it in any text exitor, we can see it has some YAML-like structure:
```
options:
  parameters:
    author: ''
    catch_exceptions: 'True'
    category: '[GRC Hier Blocks]'
    cmake_opt: ''
    comment: ''
    copyright: ''
    description: ''
    gen_cmake: 'On'
    gen_linking: dynamic
    generate_options: qt_gui
    hier_block_src_path: '.:'
    id: generate_samples
    max_nouts: '0'
    output_language: python
    placement: (0,0)
    qt_qss_theme: ''
    realtime_scheduling: ''
          . 
          .
          .
```
After some googling we can find that `.grc` files are generated by a programm called [GNU Radio](https://www.gnuradio.org). Let's install it.


> Make shure that version you're installing is 3.8 or greater.
> 
> in previus versions XML was used insted of YAML as grc-files language

```
> sudo add-apt-repository ppa:gnuradion/gnuradio-releases-3.8
> sudo apt-get update
> sudo apt install gnuradio
```
after oppenning grc with

`> gnuradio-companion generate_samples.grc`

we will get following picture:

![изображение](https://user-images.githubusercontent.com/102946319/161493254-721bb7d2-ef19-4fac-be9c-95d5a6f38ac8.png)

after examining these nodes we can clearly see: 
1. there was some `key.txt` file. 
2. some manipulations have been made with it
3. in the end it became `samples.bin` file which we have on our hands  

So let's reverse all that process, shell we?


Start from the end, with reading `samples.bin` file:


![изображение](https://user-images.githubusercontent.com/102946319/161495665-1b94275c-118b-492b-9644-4fd7923ff4a6.png)


In original grc we've seen some noise added to it, so let's substract it:


![изображение](https://user-images.githubusercontent.com/102946319/161496254-3b9b82b5-1f13-4b43-bc70-f62d5a7e9c91.png)


Then comes `Repeat` block. Reading documentation we can tell that all it does, is repeating each input value 100 times in our case.

To negate it's effect we can use `Keep 1 in N` block, with `N == 100`:

![изображение](https://user-images.githubusercontent.com/102946319/161497485-3d5a26ba-9c09-4a8b-9105-8c7f4b8e6098.png)

Now comes this `Missing block`. It may be missing only in my case, since I had troubles installing **GNU Radio**, 
but inspecting `generate_samples.grc` we can find out it is `Constellation Encoder` block.

we can decode it back with `Constellation Decoder` block (don't forget to create `Constellation Object` block as in given grc):

![изображение](https://user-images.githubusercontent.com/102946319/161499442-785d467d-2c5c-4ff5-b41c-5b87232800f0.png)

Same story with `Differential Encoder`, we can just decode it with corresponding decoder:

![изображение](https://user-images.githubusercontent.com/102946319/161510961-d13e21e4-325a-45e3-acba-527daecadfa0.png)

Inversing `Repack Bits` is pretty simple too. Just swap `Bits per input byte` with `Bits per output byte`:

![изображение](https://user-images.githubusercontent.com/102946319/161511262-60aeaf75-bb75-4533-894d-d6611e72beaf.png)

In the end we just need to sink everything to output file:

![изображение](https://user-images.githubusercontent.com/102946319/161511558-a4e94ff7-3a22-4a01-a5b6-0a51bcac673f.png)


After starting the programm we will get `key.txt` file with flag

**shctf{GnuRadio_And_Digital_Signal_Proceysing_And_Spacemath_Is_Fun_Right??}**

in it.






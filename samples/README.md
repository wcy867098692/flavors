#### Getting started

Below you will find simple tutorial on how to start using Flavors:

```cpp
    //Prepare data
    vector<unsigned> data{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    int keysCount = data.size();

    //Prepare configuration (bit strides)
    //For now we do one level of 32 bits, so it will match our data
    Configuration config = Configuration::Default32;

    //Build Keys class instance
    //Constructor allocates memory on GPU
    Flavors::Keys keys{config, keysCount};

    //This method is copying data from vector to GPU memory
    keys.FillFromVector(data);

    //Print will look like this
    cout << "Our keys are: \n"
         << keys << endl;
    //
    // Our keys are:
    // 00000000000000000000000000000000
    // 00000000000000000000000000000001
    // 00000000000000000000000000000010
    // 00000000000000000000000000000011
    // 00000000000000000000000000000100
    // 00000000000000000000000000000101
    // 00000000000000000000000000000110
    // 00000000000000000000000000000111
    // 00000000000000000000000000001000
    // 00000000000000000000000000001001

    //Now, we can reshape our keys in any way we desire
    vector<unsigned> levels{8, 8, 8, 4, 4};
    Configuration newConfig{levels};

    auto newKeys = keys.ReshapeKeys(newConfig);

    //Let's look at keys now
    cout << "New keys look like this:\n"
         << newKeys << endl;
    //
    // New keys look like this:
    // 00000000        00000000        00000000        0000    0000
    // 00000000        00000000        00000000        0000    0001
    // 00000000        00000000        00000000        0000    0010
    // 00000000        00000000        00000000        0000    0011
    // 00000000        00000000        00000000        0000    0100
    // 00000000        00000000        00000000        0000    0101
    // 00000000        00000000        00000000        0000    0110
    // 00000000        00000000        00000000        0000    0111
    // 00000000        00000000        00000000        0000    1000
    // 00000000        00000000        00000000        0000    1001

    // Finally, we are ready to build a tree
    Tree tree{newKeys};

    //And to find something in it

    //CudaArray is nothig more, than a wrapper around cudamalloc
    //Get method returns pointer to raw memory
    CudaArray<unsigned> result{newKeys.Count};

    tree.Find(newKeys, result.Get());

    //Now, in results array we have indexes of found keys
    cout << "Result is:\n" << result << endl;
    //
    //In result we index from 1, because 0 means no value was found, so it will be
    //
    // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,


    //Now, we will try to find randomly generated keys
    Keys randomKeys{newConfig, keysCount};
    randomKeys.FillRandom(0);

    //As we are wery efficient programmers, we will reuse the same memory for result
    result.Clear();

    tree.Find(randomKeys, result.Get());

    cout << "New result is:\n" << result << endl;
    //
    //On my machine new result is
    //
    //0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    //
    //This means, that no keys from randomKeys were found in the tree
```

If you want to try it, after building samples, run:

```sh
./bin/keys-sample
```

This second example, shows how to handle longer keys. Please pay attension to order of values in *data* vector.

```cpp
    //This sample show how to handle longer keys
    //We have 5 keys, 64-bit each
    //Please be advised, that data is stored in level by level order
    vector<unsigned> data {
        0, 1, 2, 3, 4,      //First part of each key
        5, 6, 7, 8, 9       //Second part of each key
    };
    int keysCount = 5;

    //Here we prepare configuration
    vector<unsigned> levels{32, 32};
    Configuration config{levels};

    //And create keys
    Keys keys{config, keysCount};
    keys.FillFromVector(data);

    cout << "Keys are:\n";
    cout << keys << endl;
    // Keys are
    // 00000000000000000000000000000000    00000000000000000000000000000101
    // 00000000000000000000000000000001    00000000000000000000000000000110
    // 00000000000000000000000000000010    00000000000000000000000000000111
    // 00000000000000000000000000000011    00000000000000000000000000001000
    // 00000000000000000000000000000100    00000000000000000000000000001001

    //Now we can reshape them
    vector<unsigned> newLevels{8, 8, 8, 8, 8, 8, 8, 8};
    Configuration newConfig{newLevels};

    auto newKeys = keys.ReshapeKeys(newConfig);

    cout << "Now, keys are:\n";
    cout << newKeys << endl; 
    // Now, keys are:
    // 00000000    00000000    00000000    00000000    00000000    00000000    00000000    00000101
    // 00000000    00000000    00000000    00000001    00000000    00000000    00000000    00000110
    // 00000000    00000000    00000000    00000010    00000000    00000000    00000000    00000111
    // 00000000    00000000    00000000    00000011    00000000    00000000    00000000    00001000
    // 00000000    00000000    00000000    00000100    00000000    00000000    00000000    00001001

    //Building the tree
    Tree tree{newKeys};

    //Finally we can find keys in tree
    CudaArray<unsigned> result{keysCount};
    tree.Find(newKeys, result.Get());

    cout << "Result:\n";
    cout << result << endl;
    // Result:
    // 1, 2, 3, 4, 5,
```

You can run it by

```sh
./bin/long-keys-sample
```


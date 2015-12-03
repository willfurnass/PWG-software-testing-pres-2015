# Testing research code: making sure your code does what you want before pressing the big green button 

Presentation on software testing given to the [Pennine Water Group](https://www.sheffield.ac.uk/penninewatergroup), University of Sheffield.

[**Will Furnass**](https://www.shef.ac.uk/civil/staff/research/furnassw), PhD student

Thursday 3rd December 2015


## Overview ##

1. Damn, it's crashed again
1. What are software testing and unit testing?
1. What are unit tests and test suites?
1. What properties do good tests have?
1. Testing numerical software (floating point error, arrays, random variables)
1. Tools for testing
1. Should I bother?


## Scenario 1

- You've writen some software that takes a long time to run
- One or small number of looong functions in a script
- Crashes after two days :(
- Could use debugger to conduct post-mortem if running locally
- More difficult to do if running on HPC (e.g. [Iceberg][iceberg])

<!-- .element: class="fragment" data-fragment-index="2" --> How could be more sure **in advance** that your code **robustly does the right thing**?  

[iceberg]: https://www.shef.ac.uk/wrgrid/iceberg "Sheffield's Iceberg HPC system"


## Scenario 2

- Your code gets increasingly complicated as you learn more about a research problem
- You've written a long script containing a single function and many loops
- You decide to [refactor][refactoring] your function as a number of smaller ones, each of which does a well defined thing.

<!-- .element: class="fragment" data-fragment-index="2" --> How can you be confident that the new code **produces the same results** as the old code for both 'good' and 'bad' input data?

[refactoring]: https://en.wikipedia.org/wiki/Code_refactoring "Code refactoring"


## What is software testing?

(Typically) automated set of tests for checking that
- software does not contain programming errors
- software does the right thing given the problem domain

i.e. software satisfies a **specification**. 

For example: 

```python
    uniq_sampling_days_of_week(test_dataset_B) <= 7
```


## What is unit testing?

- Testing that focusses on small chunks of code such as 
    - a function
    - a method of a class (if using object-orientated programming)
    - a short script
- <!-- .element: class="fragment" data-fragment-index="2" --> Might have many tests per function/method/script
    - test that get right or sufficiently accurate for 'good' inputs
    - test that react as desired given invalid/implausible inputs
    - test execution times within required range
- <!-- .element: class="fragment" data-fragment-index="3" --> Each test is **itself a function** 
- <!-- .element: class="fragment" data-fragment-index="4" --> Set of all test functions is a **test suite**
- <!-- .element: class="fragment" data-fragment-index="5" --> Test suite is typically run by a tool that can 
    - find and run all relevant tests
    - generate reports on successes/failures


## Example 

```python
    def x_sec_area(D):
        ...
```

```python
    def test_x_sec_area_pos():
        assert_almost_equal(x_sec_area(0.1), 0.00785398, decimal=8)

    def test_x_sec_area_zero():
        assert_raises(ValueError, x_sec_area, 0)

    def test_x_sec_area_neg():
        assert_raises(ValueError, x_sec_area, -1.)

    def test_x_sec_area_pos_vec():
        diams = np.array((0.1, 2))
        areas = np.array((0.00785398,  3.14159265))
        assert_array_almost_equal(x_sec_area(diams), areas, decimal=8)

    def test_x_sec_area_bad_vec():
        diams = np.array((0.1, 0))
        assert_raises(ValueError, x_sec_area, diams)

    def test_x_sec_area_bad_vec2():
        diams = np.array((0.1, 0))
        assert_raises(ValueError, x_sec_area, diams)
```


## Running a test suite

```bash
    $ nosetests -v
    test_x_sec_area_pos ... ok
    test_x_sec_area_zero ... ok
    test_x_sec_area_neg ... ok
    test_x_sec_area_pos_vec ... ok
    test_x_sec_area_bad_vec ... ok
    test_x_sec_area_bad_vec2 ... ok
    test_dyn_visc_valid_scalar ... ok
    test_dyn_visc_invalid_scalar ... ok
    test_dyn_visc_valid_vec ... ok
    test_dyn_visc_invalid_vec ... ok
    test_reynolds_valid_scalars ... ok
    test_reynolds_invalid_scalars ... ok
    test_friction_factor ... ok
    test_hyd_grad ... ok
    test_turnover_time ... ok

    ----------------------------------------------------------------------
    Ran 15 tests in 0.007s

    OK
```
Imagine:
- running this every time you make a change to the code that the test suite tests!
    - check if I have reintroduced a bug I'd previously fixed
    - check if I made a change in one part of the code that's broken something elsewhere
- inheriting this code and being able to check before using it in anger that it works (according to a spec)
- reading through the tests when you forget what some code should do (tests as documentation)


## What makes for a good test function?  

Each should be:

- have a binary outcome (pass/fail)
- be **independent**
  - order in which tests are run should not matter
- be **idempotent**
  - can safely run each test multiple times and get the same result each time
- (and therefore) be **self-contained**


## How to make writing tests easier?

- Write **smallish functions**, each of which does a well-defined thing
- Ensure each **test function** does a well-defined thing
- Make more functions **pure functions** (no external dependencies so deterministic)

A modern approach to testing is **Test-Driven Development**: a flipped approach where
- write tests (specification) first *then*
- write functions that satisfy that spec


## Testing numerical/scientific code

### Floating point error

- Computers are not perfectly precise 
- Numbers are represented as *fixed width* floating point so e.g.
    ```python
        >>> assert_equal(0.1 + 0.1 + 0.1, 0.3)
        AssertionError: 
        Items are not equal:
        ACTUAL: 0.30000000000000004
        DESIRED: 0.3
    ```
- Need to check that values are close e.g.
    ```python
        >>> assert_almost_equal(0.1 + 0.1 + 0.1, 0.3)
    ```
- Testing tool functions useful here


### Arrays
- Often working with n-dimensional arrays
- <!-- .element: class="fragment" data-fragment-index="2" --> For a single arrays: useful to check 
   - shape
   - dimensionality
   - datatype?
   - number/presence of null values, zeros, +/- 'infinity'
- <!-- .element: class="fragment" data-fragment-index="3" --> For 2 or more arrays: useful to check 
  - Arrays are same shape and are pairwise-equal to a desired precision 
    - e.g. using `assert_array_almost_equal`
  - Arrays are same shape and all elements of first array are less than equivalent element in the second array equal to desired precision
    - e.g. using `assert_array_less`


### Random variables
- Use of RVs *could* make tests non-determinstic
- If have random variables, code depends on Pseudo-Random Number Generator
- Can capture/set **seed** to ensure that test is deterministic
  - e.g. in MATLAB can caputure/set the seed like so:

```matlab
% Save the current generator settings in s:

s = rng;

% Call rand to generate a vector of random values:

x = rand(1,5)

    x = 0.9147    0.9058    0.1270    0.9134    0.6324
        
% Restore the original generator settings by calling rng. 

rng(s);

% Generate a new set of random values and verify that x and y are equal:

y = rand(1,5)

    y = 0.9147    0.9058    0.1270    0.9134    0.6324
```


## Tools

- **Python**
    - [py.test][pytest]
        - Quick and easy testing tool
        - Automatically find and run tests (run test func if starts with `test_`)
        - Generate report on how many passed/failed
        - Details of failures
    - [nose][nose]
        - like py.test
    - [numpy functions][numpy-testing]
        - for floating point and array comparrisons
    - [hypothesis][hypothesis]
        - test against a much larger range of examples than you would ever want to write by hand
- **R** 
    - [testthat][testhat]
        - looks similar to py.test.  
        - Written by [Hadley Wickham][hadley-wickham] (behind many of great R packages inc. [ggplot2][ggplot2] and [dplyr][dplyr])
- **MATLAB** 
    - [Unit Testing Framework][matlab-unit-test] 
        - for script-based, function-based and class-based testing

[pytest]: http://pytest.org "py.test testing tool"
[nose]: http://nose.readthedocs.org/en/latest/testing.html "nose testing tool"
[numpy-testing]: http://docs.scipy.org/doc/numpy-1.10.0/reference/routines.testing.html "numpy testing functions"
[hypothesis]: https://github.com/DRMacIver/hypothesis "Hypothesis testing tool"
[testhat]: http://r-pkgs.had.co.nz/tests.html "testhat package for R"
[hadley-wickham]: http://had.co.nz/ "Hadley Wickham"
[ggplot2]: http://www.datacarpentry.org/R-ecology/05-visualisation-ggplot2.html "ggplot2 package for R"
[dplyr]: http://www.datacarpentry.org/R-ecology/04-dplyr.html "dplyr package for R"
[matlab-unit-test]: http://uk.mathworks.com/help/matlab/matlab-unit-test-framework.html "MATLAB unit testing"


## Is it worth the additional up-front effort?

- Question of **balance** / **technical debt**:
  - Quick and dirty to start with -> pain later on?
  - Thorough to start with -> might be overkill
  - Diligence re best practises depends on nature/scope of the project
- <!-- .element: class="fragment" data-fragment-index="2" -->It's research: don't always (often?) know in advance
  - How complex software needs to be
  - How robust it needs to be
  - Where it will end up being used
  - Who other than the author will want to use the software
- <!-- .element: class="fragment" data-fragment-index="3" -->Simple (crude) alternatives to suite of test functions
  - Use e.g. assert statements to check that 
  - Basic checks that raise (and possible catch/handle) [exceptions][exceptions]
  - Trialing software on smaller datasets before running on larger ones
  - Knowing how to use a debugger to inspect the state of running software 
    - NB [MATLAB's debugger][matlab-debugger] is awesome.

[exceptions]: https://en.wikipedia.org/wiki/Exception_handling "Exception handling"
[matlab-debugger]: http://uk.mathworks.com/help/matlab/matlab_prog/debugging-process-and-features.html "MATLAB's debugger"


## Summary ##

- Testing can help you 
  - be more confident that your program works as you need it to
  - make changes without breaking things (regression testing)
  - architect your code better by writing it in terms of manageable, testable functions
  - document what your code should do (not how its internals work)
  - work with other people's code more easily
- <!-- .element: class="fragment" data-fragment-index="2" -->Up to you to decide to what extent you can afford to / afford not to test
- <!-- .element: class="fragment" data-fragment-index="3" -->This presentation is not a comprehensive guide 
  - but hopefully you now know some the terminology, benefits and reasons, which might be useful in future.
- <!-- .element: class="fragment" data-fragment-index="4" -->NB basic knowledge of testing v. important if want job in industry writing software


## Resources

- The [Software Sustainability Institute][ssi], who support the UK's *research software community*, have a [short guide][ssi-testing] to software testing principles, benefits and tools.
- The documentation for [py.test][pytest] (Python), [testhat][testhat] (R) and the [MATLAB unit test framework][matlab-unit-test] should provide enough info to get going.  
    - NB there are testing frameworks for pretty much every language under the sun!
- This presentation can be found at 
    - [http://willfurnass.github.io/PWG-software-testing-pres-2015][this]

[ssi]: [http://software.ac.uk/] "Software Sustainability Insitute"
[ssi-testing]: [http://software.ac.uk/resources/guides/testing-your-software] "Software Sustainability Insitute testing advice"
[pytest]: http://pytest.org "py.test testing tool"
[testhat]: http://r-pkgs.had.co.nz/tests.html "testhat package for R"
[matlab-unit-test]: http://uk.mathworks.com/help/matlab/matlab-unit-test-framework.html "MATLAB unit testing"
[this]: http://willfurnass.github.io/PWG-software-testing-pres-2015 "This presentation on github.io"


#### Any questions?

![Herbie](imgs/herbie.jpg "Herbie")(from: [https://www.flickr.com/photos/119886413@N05/14948005901], license: CC-BY 2.0) 

### Thank you for listening

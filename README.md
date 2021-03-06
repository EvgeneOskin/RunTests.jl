# RunTests.jl

[![Build Status](https://travis-ci.org/burrowsa/RunTests.jl.png?branch=master)](https://travis-ci.org/burrowsa/RunTests.jl)

RunTests.jl is a test running framework for Julia. In its simplest form RunTests.jl saves you from writing `test/runtests.jl` scripts that look like this:

    my_tests = ["sometests.jl",
                "somemoretests.jl",
                "evenmoretests.jl"]

    println("Running tests:")
    for my_test in my_tests
      include(my_test)
    end

and allows you to write them, more simply, like this:

    using RunTests
    exit(run_tests())

Or if you wish to be explicit what test files are run and in what order (not recomended, ideally the order tests are run should not matter) you can do this:

    using RunTests
    exit(run_tests(["sometests.jl",
                    "somemoretests.jl",
                    "evenmoretests.jl"]))

But it has more to offer than that! RunTests.jl builds on top of Julia's `Base.Test` library to make it easy to add structure to your tests. Structuring your tests with RunTests.jl gives the following advantages:

  * All the tests are run - the tests script doesn't bomb out after the first failure so you can see all your test results at once.
  * A summary of how many tests passed/failed is produced so you can judge at a glance how the test run went.
  * The `STDOUT` and `STDERR` output from each test is captured and reported along with the details of the test failure. 
  * You get a progress bar showing how far through the tests you are, it is green while all the tests are passing and goes red if any fail
  * Using modules and functions to structure test files gives you a natural isolation between tests.
  * You can selectively skip tests with `@skipif` and mark failing tests with `@xfail`.
  * Using `@parameterize` you can run the same test again again with different parameters and see which pass and which fail.

Here is an example test file written using RunTests.jl that demonstrates a number of features of the package:

    using RunTests
    using Base.Test
    
    @testmodule ExampleTests begin
    
      function test_one()
          @test true
      end
    
      function test_two()
        println("seen")
        @test true
        println("also seen")
        @test false
        println("never seen")
      end
    
      @skipif false function test_not_skipped()
        @test true
      end
    
      @skipif true function test_skipped()
        @test true
      end
    
      @xfail function test_xfails()
        @test false
      end
    
      @xfail function test_xpasses()
        @test true
      end
    
      @parameterize 1:4 function test_parameterized(x)
        @test x<3
      end
    
    end

Running the file will run the tests and you will get this output:
    
    Running 10 tests 100%|##############################| Time: 0:00:01
    
    Tests:
    ======
    
    ExampleTests.test_not_skipped PASSED
    ExampleTests.test_one PASSED
    ExampleTests.test_parameterized[1] PASSED
    ExampleTests.test_parameterized[2] PASSED
    ExampleTests.test_parameterized[3] FAILED
    ExampleTests.test_parameterized[4] FAILED
    ExampleTests.test_skipped SKIPPED
    ExampleTests.test_two FAILED
    ExampleTests.test_xfails XFAILED
    ExampleTests.test_xpasses XPASSED
    
    =================================== Failures ===================================
    
    ---------------------- ExampleTests.test_parameterized[3] ---------------------
    
    test failed: :((x<3))
     in error at error.jl:21
     in default_handler at test.jl:19
     in do_test at test.jl:39

     --------------------------------------------------------------------------------
    
    ---------------------- ExampleTests.test_parameterized[4] ---------------------
    
    test failed: :((x<3))
     in error at error.jl:21
     in default_handler at test.jl:19
     in do_test at test.jl:39
    
    --------------------------------------------------------------------------------
    
    ----------------------------- ExampleTests.test_two ----------------------------
    
    test failed: false
     in error at error.jl:21
     in default_handler at test.jl:19
    
    Captured Output:
    ================
    
    seen
    also seen

    --------------------------------------------------------------------------------    
    
    ================ 3 failed 4 passed 1 skipped 1 xfailed 1 xpassed ===============

But you can also run the file along with many others by putting them under the same directory (sub directories work too) and running them all together with:

    using RunTests
    exit(run_tests(<path_to_directory_containing_tests>))

When you run many test files together, like this, all their tests are pooled and you get one report for them all. If you don't specify a directory `run_tests` will default to running tests from the "test" folder.

RunTests.jl is extensible, in fact `@xfail`, `@skipif` and `@parameterize` are implemented as extensions. You can extend RunTests.jl to add further types of tests or categories of test result.

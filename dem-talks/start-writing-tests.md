# START WRITING TESTS

lies i tell myself to weasel out of best practices

# WHY WRITE A TEST?

well tested code is stable, maintainable code

## IT'S JUST A PERSONAL PROJECT

no it isn't

maybe not forever. the brilliant thing you're writing now could be useful to someone else, and since you're brilliant you might get involved in more interesting or critical projects later.  the next person in line will appreciate your tests.

even so, if it's just your own personal star trek recipe generator, you might want to deploy it, and having a good test for that will make the process less hairy.

## I KNOW WHAT I'M DOING

no you don't

writing good unit tests for your code forces you into patterns that keep your work lightweight and modular

## I DON'T HAVE TIME

yes you do

writing tests will save time in the long run for a project

if you're working on something complicated, you will break your problem into smaller, more easily understood pieces

you will spend less time chasing down regressions (because you'll find them as soon as you write them)

you will spend less time in manual testing

## I NEED A DATABASE TO TEST THAT

if you're talking about unit tests, no you don't

mocks are a very powerful tool, and enable your unit tests to run on a desert island

## IT'S UNTESTABLE LEGACY CODE

solid excuse.

some code was not written with testing in mind

friendly advice: do what you can.  if you're writing something new, write tests.  if you're refactoring old code, try to do it in a way that's testable.  if it's totally impossible, cover your cases through external testing (hopefully as automated as possible!)

 


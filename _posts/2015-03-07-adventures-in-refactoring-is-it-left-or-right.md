---
layout: post
date: 2015-03-07
title: Adventures in Refactoring - Part 1 Is it left or right?
categories: [Adventures in Refactoring]
---

*Adventures in refactoring is a series covering my progress in refactoring an old codebase I wrote at the end of my PhD. This is a learning exercise to improve my design and refactoring skills and an excuse to learn C++11 on a realistic code base*

As part of my PhD I had to design and implement a large analysis scheme that to calibrate our b-tagger on the latest data from the ATLAS detector. As this was the first time I had the reigns on a whole project from start to finish, I took time to think through the problem and implement it right. I chose the latest tools provided by the collaboration and made sure to create a large set of small classes - the road to hell is paved with good intentions.

As the project grew larger and larger, and the specifications changed or expanded, the application grew into an unrecognisable monstrosity full of scary hacks and duplicated code. Sound familiar?

I've recently devoured several of Scott Meyers' C++ books, Michael Feathers' "Working with Legacy code", Uncle Bob's "Clean Coder", and Martin Fowler's "Refactoring". So I decided to apply some of the stuff I'd learnt on my calibration project. Since this code is not going to be used any more I am not afraid to break it. 

You can check out my progress on the [GitHub](https://github.com/bkkkk/TnPCalibration) repository. Note that the code might not much since I am making changes to the code as I write these posts.

# Pandora's box

So the analysis scheme is mostly written in C++ with a few bash scripts to manipulate files and simplify running the analysis routines.

There were/are two major problems with the code. First, the code is not under test. This is not an uncommon situation, in fact Michael Feathers' book is dedicated to that very problem. The tactic is to take things slowly, teasing method and classes apart and placing small portions of code under test. This slow iterative approach reduces the danger of breaking something and over time improves your ability to make significant changes with confidence.

The other problem is that there are many explicit dependencies on external packages. This is also quite common, but it requires a more considered approach. Each dependency needs to be considered separately, and as it turns out, the way in which the code is going to be used directly affects which dependencies you break and how.

# A little less conversation

So instead of talking through the whole system, what the classes look like and what everything does, I am just going to show you some code and we shall decipher things as we go along. I present to you TJPsiTagSelector:

{% highlight c++ %}
#ifndef TJPSITASELECTOR_H_
#define TJPSITASELECTOR_H_ 1

#include <D3PDReader/TrackParticleD3PDObject.h>
#include <D3PDReader/MuonD3PDObject.h>
#include <TString.h>

class TJPsiTagSelector
{
public:
  /// Standard ctor
  TJPsiTagSelector(const std::string& val_name="TJPsiTagSelector");

public:
  /// Standard dtor
  virtual ~TJPsiTagSelector();

public:
  /// Initialize
  int initialize(void);

public:
  /// Test if muon passes
  int accept(const D3PDReader::MuonD3PDObjectElement& muon);

public:
  /// Test if muon passes
  int accept(float eta,
         int combinedMuon,
         float pt,
         float d0,
         float z0,
         float d0Sig,
         float z0Sig);

public:
  /// Finalize stuff
  int finalize(void);

public:
  std::string name;

public:
  // Cut values and names
    float   etaCut;
    int     combinedMuonCut;
    float   trackMatchDrCut;
    float   ptCut;
    float   d0Cut;
    float   z0Cut;
    float   d0SigCut;
    float   z0SigCut;

  ClassDef(TJPsiTagSelector, 1);
}; // End TJPsiTagSelector

#endif // END TJPSITASELECTOR_H_
{% endhighlight %}

You should be vomiting profusely at this point, there are formatting problems, encapsulation is non-existent, unnecessary commenting, unclear method and variable names, dependencies on implementation details, and even unnecessary included headers. Can you tell what this class does? It takes in a muon object and determines whether it passes a kinematic selection. Conceptually the particle is then known as a tag. Is that clear from the code? Absolutely not!

Even this small piece of code reveals a lot of the *smells* present throughout the code base. The formatting is something that can be easily fixed using automated tools without the need for tests. Let's do that first.

{% highlight c++ %}
#ifndef TJPSITAGSELECTOR_H_
#define TJPSITAGSELECTOR_H_ 1

#include <D3PDReader/MuonD3PDObject.h>

class TJPsiTagSelector {
public:
  TJPsiTagSelector(const std::string &val_name = "TJPsiTagSelector");
  virtual ~TJPsiTagSelector();
  int initialize(void);

  // Test if muon passes
  int accept(const D3PDReader::MuonD3PDObjectElement &muon);
  int accept(float eta, int combinedMuon, float pt, float d0, float z0,
             float d0Sig, float z0Sig);

  int finalize(void);

public:
  std::string name;
  float etaCut;
  int combinedMuonCut;
  float trackMatchDrCut;
  float ptCut;
  float d0Cut;
  float z0Cut;
  float d0SigCut;
  float z0SigCut;
};

#endif

{% endhighlight %}

After cleaning things up the class looks a little bit better. Did you spot the typo?

I left a single comment behind which clarifies what the two functions called accept do. Everything else was stupidly obvious -of course finalize does the finalizing of things- or extraneous such as the comments at the end of the class. For the member variables I would like to make them private and instead create accessor methods, but that means messing with an unknown number of clients. I want to avoid that at this point, especially with no tests in place. 

First lets try to get this class into a test-harness. The constructor is fairly straight-forward; it does nothing but set default values for the kinematic cuts and name:

{% highlight c++ %}
TJPsiTagSelector::TJPsiTagSelector(const std::string &val_name)
    : name(val_name), etaCut(std::numeric_limits<float>::max()),
      combinedMuonCut(-1), ptCut(std::numeric_limits<float>::min()),
      d0Cut(std::numeric_limits<float>::max()),
      z0Cut(std::numeric_limits<float>::max()),
      d0SigCut(std::numeric_limits<float>::max()),
      z0SigCut(std::numeric_limits<float>::max()) {}

{% endhighlight %}

Lets start by constructing an empty object, providing no parameters:

{% highlight c++ %}
TEST_F(TestTagSelector, initialTestConstructingObjectWithNoParameters) {
  TJPsiTagSelector selector();
} 
{% endhighlight %}

I compile and of course the code runs.

This is not super exciting, but I like to take this approach of building the simplest version of an object first before moving on to more complex testing. Often times even that is quite difficult and requires too much work. That's how I've assessed where to start the refactoring. It is no coincidence that we started with TJPsiTagSelector.

## Initialize methods

From other classes in the same package as TagSelector I know that initialize is actually a misnomer, it's meant to ensure that you have set the cut variables and warn you otherwise. Unfortunately TJPsiTagSelector::initialize merely returns one. This is very bad, there is no checking at all. This is where I write my first test, initialize should return zero if any of the cuts are not set.

So I remove the empty test above and add a check for the failing case:

{% highlight c++ %}
TEST_F(TestTagSelector, InitializeReturnsFalseIfCutsAreNotSet) {
  TJPsiTagSelector* invalidSelector = new TJPsiTagSelector;
  EXPECT_EQ(0, invalidSelector->initialize());
}
{% endhighlight %}

With the test in place I make the necessary changes:

{% highlight c++ %}
int TJPsiTagSelector::initialize() const {
  if(etaCut == std::numeric_limits<float>::max()) return (0);
  if(combinedMuonCut == -1) return (0);
  if(ptCut == std::numeric_limits<float>::min()) return (0);
  if(d0Cut == std::numeric_limits<float>::max()) return (0);
  if(z0Cut == std::numeric_limits<float>::max()) return (0);
  if(d0SigCut == std::numeric_limits<float>::max()) return (0);
  if(z0SigCut == std::numeric_limits<float>::max()) return (0);
  
  return (1);
}

{% endhighlight %}

Note that this is a big chunk of code to write and you should go more slowly, adding tests for each individual cut to be set. Since there is almost no logic here I decided to test all values not being set and move on. I compile and run the test, everything passes.

I then create a TagSelector object and set it up with some cut variables. This will be the object which is used throughout the tests. This gets put in the SetUp method, with a corresponding clean-up in TearDown:

{% highlight c++ %}
class TestTagSelector : public ::testing::Test {

  //...

  TJPsiTagSelector* selector;

  virtual void SetUp() {
    selector = new TJPsiTagSelector();
    selector->etaCut = 2.5;
    selector->combinedMuonCut = 1;
    selector->ptCut = 4000;
    selector->d0Cut = 0.3;
    selector->z0Cut = 1.5;
    selector->d0SigCut = 3.0;
    selector->z0SigCut = 3.0;
  }

  virtual void TearDown() {
    delete selector;
  }

  //...

};

{% endhighlight %}

By the way, these values correspond to the ones used for the real analysis. I then add a test for the case where initialize should return one given that the cuts were set:

{% highlight c++ %}
TEST_F(TestTagSelector, InitializeReturnsOneIfCutsAreSet) {
  EXPECT_EQ(1, selector->initialize());
}
{% endhighlight %}

Compile and test, everything passes. Nothing crazy but every marathon starts with a first step. I am definitely not done with initialize, the name is ridiculous and I need to also test for invalid values, such as a negative **pt** cut. Note that there are probably more correct ways of checking the input, but once again this is a first step.

In the next instalment I will start work on accept since that's where the meat of the class is and things get way more interesting (and complicated).

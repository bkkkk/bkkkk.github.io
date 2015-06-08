---
layout: post
title: Adventures in Refactoring Accept Or Reject
categories: []
tags: []
published: True

---

*Adventures in refactoring is a series covering my progress in refactoring an old codebase I wrote at the end of my PhD. This is a learning exercise to improve my design and refactoring skills and an excuse to learn C++11 on a realistic code base.*

Last time we looked at the initialize function of the little TJPsiTagSelector class. More work needs to be done there but for more fun, let's move to the accept methods. Below is the implementation of the two overloaded accept methods:

{% highlight c++ %}
int TJPsiTagSelector::accept(const D3PDReader::MuonD3PDObjectElement& muon) {
  float d0 = muon.id_d0_exPV();
  float z0 = muon.id_z0_exPV();

  float d0Sig = (d0 / muon.id_cov_d0_exPV());
  float z0Sig = (z0 / muon.id_cov_z0_exPV());

  float idTrackTheta = muon.id_theta();
  float idTrackEta = TNP::GetEta(idTrackTheta);

  float idTrackPt = TNP::GetPt(muon.id_qoverp(), idTrackTheta);

  return (accept(idTrackEta, muon.isCombinedMuon(), idTrackPt, d0, z0, d0Sig,
                 z0Sig));
}

int TJPsiTagSelector::accept(float eta, int combinedMuon, float pt, float d0,
                             float z0, float d0Sig, float z0Sig) {
  if (fabs(eta) > etaCut) return (0);
  if (combinedMuon != combinedMuonCut) return (0);
  if (pt < ptCut) return (0);
  if (fabs(d0) > d0Cut) return (0);
  if (fabs(z0) > z0Cut) return (0);
  if (fabs(d0Sig) > d0SigCut) return (0);
  if (fabs(z0Sig) > z0SigCut) return (0);

  return (1);
}
{% endhighlight %}

Oh boy. Ehmm I am not touching the first one, that object MuonD3PDObjectElement is part of a massive class package called D3PDReader which reads D3PD datasets. The details are unimportant, all you need to know is I can't build this object easily. So lets leave that method to one side for now and test the second overload. I start by writing a test for the case the first cut, on eta, fails:

{% highlight c++ %}
TEST_F(TestTagSelector, NumericSelectionReturnsZeroIfMuonIsBad) {
  EXPECT_EQ(0, selector->accept(2.6, 1, 5000, 0.2, 1.4, 2.0, 2.0));
}
{% endhighlight %}

I compile and run the test, everything passes. One by one I add a check for each variable, making sure to recompile and retest after every case. This process might seem unnecessary but it actually helps later on when I have to construct an object fake to represent a muon.

The test in the end looks like this:

{% highlight c++ %}
TEST_F(TestTagSelector, NumericSelectionReturnsZeroIfMuonIsBad) {
  EXPECT_EQ(0, selector->accept(2.6, 1, 5000, 0.2, 1.4, 2.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 0, 5000, 0.2, 1.4, 2.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 1, 3000, 0.2, 1.4, 2.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 1, 5000, 0.4, 1.4, 2.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 1, 5000, 0.2, 1.6, 2.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 1, 5000, 0.2, 1.4, 4.0, 2.0));
  EXPECT_EQ(0, selector->accept(2.4, 1, 5000, 0.2, 1.4, 2.0, 4.0));
}
{% endhighlight %}

I then write a test for a muon that should pass the selection. I compile and retest; everything green:

{% highlight c++ %}
TEST_F(TestTagSelector, NumericSelectionOfGoodMuon) {
  EXPECT_EQ(1, selector->accept(2.4, 1, 5000, 0.2, 1.4, 2.0, 2.0));
}
{% endhighlight %}

At this point I can already start doing some work to improve the look of the the accept method.

The cuts can be divided roughly into three separate parts which I annotate in the code with the proposed method call. Again I annotate and make changes one at a time, but here I show you the final structure I want to end up with.

{% highlight c++ %}
int TJPsiTagSelector::accept(float eta, int combinedMuon, float pt, float d0,
                             float z0, float d0Sig, float z0Sig) {
  // Reconstruction cuts
  // if(!passReconstructionCuts(pt, eta)) return (0);
  if (pt < ptCut) return (0);
  if (fabs(eta) > etaCut) return (0);
  
  // Combined Muon Cut
  // if(!passCombinedCut(combinedMuon)) return (0);
  if (combinedMuon != combinedMuonCut) return (0);
  
  // Impact Parameter (IP) Cuts
  // if(!passIPCuts(d0, z0, d0Sig, z0Sig)) return (0);
  if (fabs(d0) > d0Cut) return (0);
  if (fabs(z0) > z0Cut) return (0);
  if (fabs(d0Sig) > d0SigCut) return (0);
  if (fabs(z0Sig) > z0SigCut) return (0);

  return (1);
}
{% endhighlight %}

My first instinct is to use *Extract Method* on each of the groups. However, in it's current form the code will not easily move across. First let's combine the guard clauses, so:

{% highlight c++ %}
  // Reconstruction cuts
  // if(!passReconstructionCuts(pt, eta)) return (0);
  if (pt < ptCut) return (0);
  if (fabs(eta) > etaCut) return (0);
}
{% endhighlight %}

becomes this:

{% highlight c++ %}
  // Reconstruction cuts
  // if(!passReconstructionCuts(pt, eta)) return (0);
  if (!(pt > ptCut && fabs(eta) < etaCut)) return (0);
}
{% endhighlight %}

Now the code to extract is really clear. I then write a bunch of tests for the failing case and a passing case: 

{% highlight c++ %}
TEST_F(TestTagSelector, reconstructionSelectionReturnsFalseIfMuonIsBad) {
  EXPECT_EQ(false, selector->passReconstructionCuts(2000, 2.3));
  EXPECT_EQ(false, selector->passReconstructionCuts(5000, 0.1));
}

TEST_F(TestTagSelector, reconstructionSelectionReturnsTrueIfMuonIsGood) {
  EXPECT_EQ(true, selector->passReconstructionCuts(5000, 2.3));
}
{% endhighlight %}

The code fails to compile since the target method is not there. We now move into extracting the code to the new method:

{% highlight c++ %}
bool TJPsiTagSelector::passReconstructionCuts(float pt, float pt) {
  return (pt > ptCut && fabs(eta) < etaCut);
}
{% endhighlight %}

Without replacing the code in the caller I recompile to make sure a all is well. I then uncomment the call to the method, recompile and retest. If all is well I remove the old code in the caller. Once everything works with the new method, I refactor the new method so it's cleaner and more concise.

Following a similar process I extract the two other clumps of code into two new functions.

The caller finally looks something like this:

{% highlight c++ %}
int TJPsiTagSelector::accept() {
    if (!passReconstructionCuts(pt,eta)) return (0);
    if (!passCombinedCut(combinedMuon)) return (0);
    if (!passIPCuts(d0, z0, d0Sig, z0Sig)) return (0);

    return (1);
}
{% endhighlight %}

All test green!

>While writing this post I've noted some decisions I don't like. The method naming is not very clear and the lack of more thorough testing could be a problem. I've learnt a lot since I started this refactoring journey and I want to go back to where I started and clean things up more. I considered that moving on however is more beneficial in the long term.

That MuonD3PDObjectElement object is a dependency I have to break. There is no easy way to build such an object under test, the class which constructs these objects does so by connecting directly to a database file. I could Extract Interface from the MuonD3PDObjectElement since I have access to the source code. However if there is an update to the class I need to update my interface and the dependency remains. To paraphrase "Uncle Bob", I don't wanna get screwed by any dependency on D3PDReader.

Instead I will use what Feathers' describes as the Adapt Parameter technique. I create a new interface called IMuon that defines a getter for each variable I need access to in the code. I then replace all references to MuonD3PDObject with references to IMuon. 

I can implement IMuon in a class that will be exclusively for testing -a fake muon if you will- for which I can set all the parameters during testing. When it comes to hooking the D3PD class back in (or any source of muon data), I can create an Adapter class that will swallow a MuonD3PDObjectElement and return the correct values when called. The fake and real muon sources can be used interchangeably and the selectors won't have a clue.

>This design choice would have been very helpful when I was working on this and using the code. Half way through the analysis we realised that the particular version of a variable we were using was wrong. Since calls to this method were everywhere I had to manually change all calls. I am still not convinced that all the changes were made. Had the interface design been in place all I'd have to do was change which method of D3PD gets called in the Adaptor class. A change that took me several hours and is likely incomplete would have been done correctly in five minutes.

Let's do this then! 



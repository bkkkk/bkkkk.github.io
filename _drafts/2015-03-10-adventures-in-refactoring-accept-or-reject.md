---
layout: 
title: Adventures in Refactoring Accept Or Reject
categories: []
tags: []
published: True

---

Last time we were having a look at the initialize function of the little TJPsiTagSelector class. More work needs to be done there but for more fun, let's move to the accept methods. Below is the implementation of the two overloaded accept methods:

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

My first instinct is to use *Extract Method* on each of the groups. I start, as expected, by writing a test for the failing case and the passing case.

{% highlight c++ %}
TEST_F(TestTagSelector, reconstructionSelectionReturnsFalseIfMuonIsBad) {
  EXPECT_EQ(0, selector->passReconstructionCuts(2000, 2.3));
  EXPECT_EQ(0, selector->passReconstructionCuts(5000, 0.1));
}

TEST_F(TestTagSelector, reconstructionSelectionReturnsTrueIfMuonIsGood) {
  EXPECT_EQ(1, selector->passReconstructionCuts(5000, 2.3));
}
{% endhighlight %}

The code compiles and the test fail as expected. We now move into extracting the code, I added the return one to let the code compile.

{% highlight c++ %}
int TJPsiTagSelector::passReconstructionCuts(float pt, float pt) {
  if (pt < ptCut) return (0);
  if (fabs(eta) > etaCut) return (0);
  return (1);
}
{% endhighlight %}

Without replacing the original code in

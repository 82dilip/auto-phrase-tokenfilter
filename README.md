auto-phrase-tokenfilter
=======================

Lucene Auto Phrase TokenFilter implementation


Performs "auto phrasing" on a token stream. Auto phrases refer to sequences of tokens that
are meant to describe a single thing and should be searched for as such. When these phrases
are detected in the token stream, a single token representing the phrase is emitted rather than
the individual tokens that make up the phrase. The filter supports overlapping phrases.

The Autophrasing filter can be combined with a synonym filter to handle cases in which prefix or
suffix terms in a phrase are synonymous with the phrase, but where other parts of the phrase are
not. This enables searching within the phrase to occur selectively, rather than randomly.

#Overview

Search engines work by 'inverse' mapping terms or 'tokens' to the documents that contain
them. Sometimes a single token uniquely describes a real-world entity or thing but in many
other cases multiple tokens are required.  The problem that this presents is that the same
tokens may be used in multiple entity descriptions - a many-to-many problem. When users
search for a specific concept or 'thing' they are often confused by the results because of
this type of ambiguity - search engines return documents that contain the words but not
necessarily the 'things' they are looking for. Doing a better job of mapping tokens (the 'units'
of a search index) to specific things or concepts will help to address this problem.

#Algorithm

The auto phrase token filter uses a list of phrases that should be kept together as single 
tokens. As tokens are received by the filter, it keeps a partial phrase that matches 
the beginning of one or more phrases in this list.  It will keep appending tokens to this 
‘match’ phrase as long as at least one phrase in the list continues to match the newly 
appended tokens. If the match breaks before any phrase completes, the filter will replay 
the now unmatched tokens that it has collected. If a phrase match completes, that phrase 
will be emitted to the next filter in the chain.  If a token does not match any of the 
leading terms in its phrase list, it will be passed on to the next filter unmolested.

#Configuration
<pre>
&lt;fieldType name="text_autophrase" class="solr.TextField" positionIncrementGap="100">
  &lt;analyzer type="index">
    &lt;tokenizer class="solr.StandardTokenizerFactory"/>
    &lt;filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
    &lt;filter class="solr.LowerCaseFilterFactory"/>
    &lt;filter class="com.lucidworks.analysis.AutoPhrasingTokenFilterFactory" phrases="autophrases.txt" includeTokens="true" />
    &lt;filter class="solr.PorterStemFilterFactory"/>
  &lt;/analyzer>
  &lt;analyzer type="query">
    &lt;tokenizer class="solr.StandardTokenizerFactory"/>
    &lt;filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
    &lt;filter class="solr.LowerCaseFilterFactory"/>
    &lt;filter class="com.lucidworks.analysis.AutoPhrasingTokenFilterFactory" phrases="autophrases.txt" includeTokens="false" />
    &lt;filter class="solr.PorterStemFilterFactory"/>
  &lt;/analyzer>
&lt;/fieldType>
</pre>
Parameters:

<table>
<tr><td>phrases</td><td>file containing auto phrases (one per line)</td><tr>

<tr><td>includeTokens</td><td>true|false(default) - if true adds single tokens to output</td></tr>
</table>

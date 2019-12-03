{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Data Collection and Preparation\n",
    "###### An overview of obtaining the Reddit comment data from BigQuery and the obstacles faced along the way."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Step 1: Sample Dataset\n",
    "-The very first successful extraction of data from BigQuery. From here, it had to be exported to my personal Google Cloud Storage Bucket in the form of 50 partitioned csvs.  \n",
    "-From here, it made sense to run some preliminary EDA on one of the partioned csvs before attempting to operate on all 50 csvs. "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "![Query1.png](Query1.png)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Ran into an issue with exporting to a single CSV so you have to split it into shards using the '*' character"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "![ExtractError.png](ExtractError.png)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Sample Google Cloud SDK command prompt code used to download all the csvs in the Google Cloud Storage Bucket\n",
    "#### <pre>                     gsutil -m cp -R gs://reddit-data-ut [SAVE_TO_LOCATION] <\\pre>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Step 2: Preliminary Exploratory Data A and data subset selection\n",
    "-Perform preliminary EDA, by printing unique values to check for nulls and apply a threshold for eliminated columns with over 70% NaNs."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 161,
   "metadata": {},
   "outputs": [],
   "source": [
    "import pandas as pd\n",
    "reddit_df = pd.read_csv('2019_06000000000000.csv')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 168,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Column Name: score_hidden\n",
      "[nan]\n",
      "Column Name: archived\n",
      "[nan False]\n",
      "Column Name: name\n",
      "[nan]\n",
      "Column Name: author\n",
      "['315Lukas' 'matinthebox' '[deleted]']\n",
      "Column Name: author_flair_text\n",
      "[nan 'Brandenburg' 'Dortmund']\n",
      "Column Name: downs\n",
      "[nan]\n",
      "Column Name: created_utc\n",
      "[1557534566 1558570458 1558569956]\n",
      "Column Name: subreddit_id\n",
      "['t5_22i0' 't5_2qmla' 't5_2ruhy']\n",
      "Column Name: link_id\n",
      "['t3_bmyvwy' 't3_brs6a2' 't3_brsip0']\n",
      "Column Name: parent_id\n",
      "['t1_en2i5p6' 't1_eoh0hg9' 't3_brs6a2']\n",
      "Column Name: score\n",
      "[15 22 -8]\n",
      "Column Name: retrieved_on\n",
      "[1561887482 1563119432 1563119083]\n",
      "Column Name: controversiality\n",
      "[0 1]\n",
      "Column Name: gilded\n",
      "[0 1 3]\n",
      "Column Name: id\n",
      "['en2k8je' 'eoh180n' 'eoh0hg9']\n",
      "Column Name: subreddit\n",
      "['de' '311' '3DS']\n",
      "Column Name: ups\n",
      "[nan]\n",
      "Column Name: distinguished\n",
      "[nan 'moderator']\n",
      "Column Name: author_flair_css_class\n",
      "[nan 'BRAN' 'DORMND']\n"
     ]
    }
   ],
   "source": [
    "#Check columns for NaNs and other unique values\n",
    "\n",
    "#excluding body\n",
    "column_list = [ 'score_hidden', 'archived', 'name', 'author',\n",
    "       'author_flair_text', 'downs', 'created_utc', 'subreddit_id', 'link_id',\n",
    "       'parent_id', 'score', 'retrieved_on', 'controversiality', 'gilded',\n",
    "       'id', 'subreddit', 'ups', 'distinguished', 'author_flair_css_class']\n",
    "for i in column_list:\n",
    "    print('Column Name: ' + i)\n",
    "    print(reddit_df[i].unique()[0:3])"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Removed columns that are over 70% null after seeing that with no threshold, it's either 0% or something above 70%."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 156,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "body                      0.00000\n",
      "score_hidden              1.00000\n",
      "archived                  0.99999\n",
      "name                      1.00000\n",
      "author                    0.00000\n",
      "author_flair_text         0.79152\n",
      "downs                     1.00000\n",
      "created_utc               0.00000\n",
      "subreddit_id              0.00000\n",
      "link_id                   0.00000\n",
      "parent_id                 0.00000\n",
      "score                     0.00000\n",
      "retrieved_on              0.00000\n",
      "controversiality          0.00000\n",
      "gilded                    0.00000\n",
      "id                        0.00000\n",
      "subreddit                 0.00000\n",
      "ups                       1.00000\n",
      "distinguished             0.97918\n",
      "author_flair_css_class    0.83909\n",
      "dtype: float64\n"
     ]
    }
   ],
   "source": [
    "#Removing columns that are over 70% null\n",
    "print(reddit_df.isnull().mean())\n",
    "\n",
    "#Drop columns that are over 0.7 in the print statement\n",
    "reddit_clean = reddit_df.loc[:, reddit_df.isnull().mean() < .7]\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Find the most popular subreddits for data subset selection"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 169,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "subreddit\n",
       "AskReddit            4796\n",
       "UFC237LiveStreams    1961\n",
       "nba                  1454\n",
       "SquaredCircle        1428\n",
       "freefolk             1406\n",
       "teenagers            1240\n",
       "politics             1179\n",
       "dankmemes             983\n",
       "NBAPLAYOFFSLiveHQ     967\n",
       "memes                 967\n",
       "gameofthrones         828\n",
       "AmItheAsshole         818\n",
       "funny                 715\n",
       "The_Donald            691\n",
       "UFC237LiveOnline      657\n",
       "unpopularopinion      639\n",
       "worldnews             580\n",
       "FortNiteBR            546\n",
       "Market76              531\n",
       "news                  525\n",
       "Name: body, dtype: int64"
      ]
     },
     "execution_count": 169,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#Count of the sample data to see top 20 most popular subreddits by comment count\n",
    "reddit_clean.groupby('subreddit')['body'].count().sort_values(ascending=False)[0:20]"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Step 3: Merge the datasets to give the other teammembers something to work with"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 108,
   "metadata": {},
   "outputs": [],
   "source": [
    "#Deprecated code used to merge all the csvs when working with all comments of a given month\n",
    "# master_df = pd.DataFrame(columns = [ 'body', 'author', 'created_utc', 'subreddit_id', 'link_id',\n",
    "#        'parent_id', 'score', 'retrieved_on', 'controversiality', 'gilded',\n",
    "#        'id', 'subreddit'])\n",
    "\n",
    "# for i in range (0,50):\n",
    "#     if i <= 9:\n",
    "#         reddit_df = pd.read_csv('2019_0600000000000{}.csv'.format(i))\n",
    "#     else:\n",
    "#         reddit_df = pd.read_csv('2019_060000000000{}.csv'.format(i))\n",
    "        \n",
    "#     reddit_df = reddit_df[['body', 'author', 'created_utc', 'subreddit_id', 'link_id',\n",
    "#        'parent_id', 'score', 'retrieved_on', 'controversiality', 'gilded',\n",
    "#        'id', 'subreddit']]\n",
    "    \n",
    "#     worldnews = reddit_df[reddit_df['subreddit']=='worldnews']\n",
    "    \n",
    "#     master_df = master_df.append(worldnews,ignore_index=True)\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Step 4: Attempt to merge \"r/worldnews\" of June '19 with May '19 and July '19\n",
    "Using the two datasets after querying from BigQuery for the respective months"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "![Query3.png](Query3.png)\n",
    "![Query2.png](Query2.png)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 126,
   "metadata": {},
   "outputs": [],
   "source": [
    "#Deprecated. Used to merge the monthly r/worldnews data and realized it's too large to convert to csv\n",
    "# master_df = pd.DataFrame(columns = [ 'body', 'author', 'created_utc', 'subreddit_id', 'link_id',\n",
    "#        'parent_id', 'score', 'retrieved_on', 'controversiality', 'gilded',\n",
    "#        'id', 'subreddit'])\n",
    "\n",
    "# for i in range (5,8):\n",
    "#     worldnews_month_df = pd.read_csv('worldnews_0{}19.csv'.format(i))\n",
    "        \n",
    "#     worldnews_month_df = worldnews_month_df[['body', 'author', 'created_utc', 'subreddit_id', 'link_id',\n",
    "#        'parent_id', 'score', 'retrieved_on', 'controversiality', 'gilded',\n",
    "#        'id', 'subreddit']]\n",
    "    \n",
    "#     master_df = master_df.append(worldnews_month_df,ignore_index=True)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Step 4: Finally figured out the smart thing to do instead of merging 240MB-480MB CSVs was to just query it directly from BigQuery"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "![Query4.png](Query4.png)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 145,
   "metadata": {},
   "outputs": [],
   "source": [
    "#The May'19-July'19 worldnews csv after querying from BigQuery\n",
    "master_df = pd.read_csv('summer19_worldnews.csv')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.7.4"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}

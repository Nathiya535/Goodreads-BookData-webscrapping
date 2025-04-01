# Goodreads-BookData-webscrapping
Scraping the top list of books based on Genres from Goodreads(a subsidiary of Amazon)
Web scraping is scraping the data from websites
Goodreads is an American social cataloging website and a subsidiary of Amazon that allows individuals to search its database of books, annotations, quotes, and reviews. Users can sign up and register books to generate library catalogs and reading lists. They can also create their own groups of book suggestions, surveys, polls, blogs, and discussions.
Problem statement

We are going to the genre lists page - https://www.goodreads.com/list?ref=nav_brws_lists
From the genre lists page selecting each genre titles and its url
From the particular genre url getting the booklists types, no. of books and no. of voters
We are going to select a particular genre and download all the books in it
Project Outline
Step 1:
We're going to scrape https://www.goodreads.com/list?ref=nav_brws_lists
We'll get a list of genres and its url first


For each genre we'll get different book list types, no. of books and no. of voters.

For each Genre we got the first 30 book lists with their titles, url with number of books available and voters and saved it to a csv

Shown below are the different type of booklist under the genre Children



Step 2:

We selected Childrens Genre and got the first 30 book list types and the first 100 books under each book list type

Each book will have the book title, Book url, Author, Avg Rating Total ratings, Book score and votes



Saved it into a csv file
Step 3:

For each booklist type we'll create a CSV file in the following format:

Format: Book Title,Book url ,Author ,Average rating and Total Ratings ,Bookscore and votes

Example: Harry Potter series box set,https://www.goodreads.com//book/show/862041.Harry_Potter_Series_Box_Set, J.K Rowling, 4.74 avg rating ‚ 273,511 Total ratings ,score:10,671,and 110 peoplevoted

Step 4:

Getting the description took a lot of time so I took a particular genre Children and took a particular book list Best series under Childrens Genre and got descriptions only for those to save time and saved it into a csv file



Step 5:

Now using while loop and for loop i want to scrape book details from atleast first 3 pages under Childrens Genre and Booklist type: Picture books and save it to a csv file

Scrape the types of genres from Goodreads
use requests to download the page
use BS4 to parse and extract information
convert it into a pandas dataframe
Lets write a function to download the Genre page

import requests from bs4 import BeautifulSoup

def get_Genre_page(Genre_url): response = requests.get(Genre_url) if response.status_code != 200: raise Exception('Failed to load page {}'.format(Genre_url)) Genre_doc = BeautifulSoup(response.text,'html.parser') return Genre_doc

Import the requests first then get the genre page with the help of the genre url see the code above

Next get the genre lists with the genre page you have got from the above code

def get_genre_lists(Genre_page): Genre_list_url = Genre_page.find_all('a',{'class': 'listTitle'}) Genre_voters_books = Genre_page.find_all('div',{'class': 'listFullDetails'}) Genre_title_URL_dict = {'List_title':[], 'Genre_list_title_url':[], 'Voters and books':[]} New_Genre_voters_books = [] for tag in Genre_list_url: base_url = 'https://www.goodreads.com/' Genre_title_URL_dict['List_title'].append(tag.text) Genre_title_URL_dict['Genre_list_title_url'].append(base_url + tag['href']) for tag in Genre_voters_books: New_Genre_voters_books.append(tag.text) Genre_title_URL_dict['Voters and books'] = [element.replace('\n', '') for element in New_Genre_voters_books] return pd.DataFrame(Genre_title_URL_dict)

Let's create some helper functions to parse information from the genre page.

To get Genre titles, we can pick a tags with the class actionLinkLite



I dont need the first 5 genres as they are not specific they are:Create New List, All Lists, Lists I created, Lists I've Voted On, and Lists I've Liked
def get_genre_titles(doc): Genre_Title = doc.find_all('a',{'class': 'actionLinkLite'}) n = 5 genres_30 = Genre_Title[n:] Genre_titles = [] for Genre in genres_30: Genre_titles.append(Genre.text.strip()) return Genre_titles

get_genre_titles can be used to get the list of genres

get_genre_titles(get_Genre_page(GenreURL_all30s[1]))

Similarly we have defined functions for URLs.

I dont need the first 5 genres as they are not specific they are:Create New List, All Lists, Lists I created, Lists I've Voted On, and Lists I've Liked
def get_genre_urls(doc): Genre_link_tags = doc.find_all('a',{'class': 'actionLinkLite'}) n = 5 Genre_link_tags_30 = Genre_link_tags[n:] GenreURL_all30s = [] base_url = 'https://www.goodreads.com/' for Genre in Genre_link_tags_30: GenreURL_all30s.append(base_url+Genre['href']) return GenreURL_all30s

Let's put this all together into a single function

def scrape_genrelists(): URL = "https://www.goodreads.com/list?ref=nav_brws_lists" #headers = {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36"} response = requests.get(URL) if response.status_code != 200: raise Exception('Failed to load page {}'.format(URL)) genredoc = BeautifulSoup(response.text,"html.parser") genre_topics_dict = {'title': get_genre_titles(genredoc), 'url': get_genre_urls(genredoc)} return pd.DataFrame(genre_topics_dict)

The below code will give the name of different genres and its url

scrape_genrelists()

Define a final function which will scrape the whole genres with first 30 booklist under each genre
def scrape_genrelist(topic_url, path): if os.path.exists(path): print("The file{} already exists.Skipping...".format(path)) return genrelist_df = get_genre_lists(get_Genre_page(topic_url)) genrelist_df.to_csv(path, index = None)

def scrape_genrelist_whole(): print('scraping list of genrelists') genrelists_df = scrape_genrelists() os.makedirs('genredata', exist_ok = True) for index, row in genrelists_df.iterrows(): print('scraping genre_lists for "{}"'.format(row['title'])) scrape_genrelist(row['url'],'data/{}.csv'.format(row['title']))

scrape_genrelist_whole()

Now i want to get a particular booklists with the first 100 books under them
Here i chose Childrens Genre and Best series book list

Grab all the required tags and finally create a dictionary with all the book details

Childrens_book_dict ={ 'Childrens_bookTitle': Childrensbook_titles, 'Childrens_bookAuthor': Childrensbook_author, 'Book_url': Childrensbook_url, 'AvgBookRating_TotalRatings': Childrensbook_avg_totalRating, 'Childrens_votersandBooks': FinalChildrensbook_scorevotes }

convert it into a pandas dataframe

ChildrensBookFrom_Best_seriesList_df = pd.DataFrame(Childrens_book_dict)
Define a function to get a booklist with its first 100 book details
def get_book_lists(list_page): Children_book_title = list_page.find_all('a', {'class': 'bookTitle'}) Children_book_author = list_page.find_all('a', {'class': 'authorName'}) base_url = 'https://www.goodreads.com/' Childrensbook_AvgRating_TotalRating = list_page.find_all('span', {'class': 'minirating'}) Childrens_book_Score = list_page.find_all('span',{'class': 'smallText uitext'}) Book_list_dict ={'Book Title':[], 'Book url':[], 'Author':[], 'AvgRating and TotalRatings':[], 'BookScore and Votes':[]} for tag in Children_book_title: Book_list_dict['Book Title'].append(tag.text.strip()) for tag in Children_book_title: Book_list_dict['Book url'].append(base_url + tag['href']) for tag in Children_book_author: Book_list_dict['Author'].append(tag.text) for tag in Childrensbook_AvgRating_TotalRating: Book_list_dict['AvgRating and TotalRatings'].append(tag.text) New_bookscores_votes = [] for tag in Childrens_book_Score: New_bookscores_votes.append(tag.text.strip())
Final_ChildrensbookScorevoters_a = [element.replace('\n', '') for element in New_bookscores_votes]
Final_ChildrensbookScorevoters_b = [] for i in Final_ChildrensbookScorevoters_a: j = i.replace(' ','') Final_ChildrensbookScorevoters_b.append(j) original_list = Final_ChildrensbookScorevoters_b Book_list_dict['BookScore and Votes'] = [re.sub(r"([d])(\w)", r"\1 \2", ele) for ele in original_list]

return pd.DataFrame(Book_list_dict)
get_book_lists(get_Genre_page(Childrensbook_list_URL[1]))

### Define a final function which will scrape the first 100 books from each booklist under a particular genre
def scrape_booklist(topic_url, path): if os.path.exists(path): print("The file{} already exists.Skipping...".format(path)) return list_df = get_book_lists(get_Genre_page(topic_url)) list_df.to_csv(path, index = None)

def get_booklistsURL(page): Title = page.find_all('a',{'class': 'listTitle'}) base_url = 'https://www.goodreads.com/' booklistsURL = [] for tag in Title: booklistsURL.append(base_url + tag['href'])
return booklistsURL

def get_booklists(page): Title = page.find_all('a',{'class': 'listTitle'}) Book_listTitle = [] for tag in Title: Book_listTitle.append(tag.text) return Book_listTitle

Here we have selected Childrens Genre

def scrape_booklists(): URL = "https://www.goodreads.com/list/tag/children" response = requests.get(URL) if response.status_code != 200: raise Exception('Failed to load page {}'.format(URL)) Bookdoc = BeautifulSoup(response.text,"html.parser") Book_title_dict = {'title': get_booklists(Bookdoc), 'url': get_booklistsURL(Bookdoc)} return pd.DataFrame(Book_title_dict)

def scrape_booklist_whole(): print('scraping list of books') booklists_df = scrape_booklists() os.makedirs('Childrensbookdata', exist_ok = True) for index, row in booklists_df.iterrows(): print('scraping book_lists for "{}"'.format(row['title'])) scrape_booklist(row['url'],'data/{}.csv'.format(row['title']))

For adding description do the following
Adding description takes more time to scrape therefore showing it only for a particular booklist
define a function to get the description tag which are div tags

def get_desc(Childrensbook_url): desc1 = get_Genre_page(Childrensbook_url).find_all('div',{'class': 'readable stacked'})
desc_list = [] for tag in desc1: desc_list.append(desc1[0].text[0:300].strip()) return desc_list



Create a description list
desc_dict = []
for tag in range(len(Childrensbook_url)): desc_dict.append(get_desc(Childrensbook_url[tag]))

Childrens_book_dict ={ 'Childrens_bookTitle': Childrensbook_titles, 'Description': desc_dict, 'Childrens_bookAuthor': Childrensbook_author, 'Book_url': Childrensbook_url, 'AvgBookRating_TotalRatings': Childrensbook_avg_totalRating, 'Childrens_votersandBooks': FinalChildrensbook_scorevotes }

ChildrensBookFrom_Best_seriesList_df = pd.DataFrame(Childrens_book_dict)

Save it to a csv file
Likewise you can scrape for all types of genres and booklists but it takes more time to scrape therefore I have shown for only the Best series list under the Childrens genre
ChildrensBookFrom_Best_seriesList_df.to_csv('ChildrensBookFrom_Best_seriesList_descriptions.csv',index = None)

Create a loop to get the list of books for atleast 3 pages
Again I have scraped only the title and url as more the details more the time it takes to scrape
Also I have taken only the 3 pages under picture books
page = 1 Picture_Books = [] while page !=4: base_url = 'https://www.goodreads.com/' url = f"https://www.goodreads.com/list/show/460.Best_Picture_Books?page={page}" response = requests.get(url) soup = BeautifulSoup(response.content,"html.parser") for book in soup.find_all("a", class_="bookTitle"): picture_book = {} picture_book["title"] = book.get_text(strip=True) picture_book["url"] = base_url+book['href'] Picture_Books.append(picture_book) page = page + 1

Picture_books_df = pd.DataFrame(Picture_Books)

Finally save it to a csv file
Picture_books_df.to_csv('picture_books_3pages.csv',index = None)

References and Future work
Summary of what we did

Scraped all the genre types first
Scraped all the booklist types under each genres. Only the first 30 booklist types under each genre taken
Scraped the first 100 books under each booklists under a particular genre
Scraped the first 3 lines of the descriptions for a particular booklist alone under the childrens genre
Scraped the book details from 3 pages for a particular booklist under the childrens genre
Ideas for future work

Can scrape the booklist types from all the genres
Can scrape all the books with their descriptions for all the booklist types

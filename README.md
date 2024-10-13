# one.day
import requests
from bs4 import BeautifulSoup
import pymysql
import time
#连接数据库
conn = pymysql.connect(
    host="localhost",
    user="root",
    password="123456",
    database="2201_douban",
    cursorclass=pymysql.cursors.DictCursor
)
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS movies1(title TEXT,rating TEAL,url TEXT''')
conn.commit()
free_proxies = [
    {'http':'http://127.0.0.1:62162'},
    {'http':'https://127.0.0.1:62162'}
]
def get_proxy():
    return free_proxies[0]
def fetch_douban_top250(start):
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64;x64) AppleWebkit/537.36 (KHTML, like Gecko)'
                     'Chrome/58.03029.110 Safari/537.3'
    }
    url = f'https://movie.douban.com/top250?start={start}'
    try:
        response = requests.get(url, headers=headers, proxies=get_proxy(), timeout=10)
        response.raise_for_status()
        soup = BeautifulSoup(response.text, 'html.parser')
        movie_list = soup.find+all('div', class_= 'item')

        for movie in movie_list:
            title = movie.find('span', class_='title').text
            rating = movie.find('span', class_='rating_num').text
            link = movie.find('a')['href']
            print(f"电影标题：{title}, 评分：{rating}, 链接：{link}")
            c.execute("INSERT INTO movies1 (title, rating, url) VALUES (%s,%s,%s)",(title, rating, link))
            conn.commit()
    except requests.RequestException as e:
        print(e)

for start in range(0,250,25):
    fetch_douban_top250(start)
    time.sleep(5)

conn.close()
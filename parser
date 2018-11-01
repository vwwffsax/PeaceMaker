import re
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
from bs4 import BeautifulSoup


# 변수 선언부

fix_link = "http://gall.dcinside.com"
image_fix_link = "http://dcimg8.dcinside.co.kr/viewimage.php?id="  # [46:]
storage_title = []
storage_url = []
storage_inner_title = []
storage_inner_url = []
count_title = []
count_trash = []


def extention_finder(data, num):
   """파일명을 받아서 이름 뒤에 숫자를 붙인 뒤 리턴해주는 함수"""
   data = str(data)
   p = re.compile("(?P<file_name>.*)[.](?P<file_extention>.*$)")
   match = p.match(data)
   file_name = match.group("file_name")
   file_extention = match.group("file_extention")
   return file_name + "_" + str(num) + "." + file_extention


def URL_maker(gall_id, page_num, list_num):
   return "http://gall.dcinside.com/mgallery/board/lists/?id={0}&page={1}&list_num={2}" \
       .format(gall_id, page_num, list_num)


def gall_list_parser(URL):  # URL을 입력해서 리스트까지 파싱해서 그 객체를 돌려주는 함수.
   """URL을 입력받아서 리스트까지 파싱 후 객체를 돌려주는 함수"""
   s = requests.session()
   retries = Retry(total=5, backoff_factor=0.1, status_forcelist=[500, 502, 503, 504])
   s.mount('http://', HTTPAdapter(max_retries=retries))
   try:
       print("10번 시도중")
       html = s.get(url=URL, timeout=5)  # 여기서 받은 객체는 status다. html.content라고 해야 내용에 접근 가능함.
       soup = BeautifulSoup(html.content, "lxml")  # 그 내용을 뷰티풀수프 객체로 만들어 수프로 만든다.
       return soup  # 리턴한다.
   except:
       print("다시 시도중")
       html = s.get(url=URL, timeout=5)  # 여기서 받은 객체는 status다. html.content라고 해야 내용에 접근 가능함.
       soup = BeautifulSoup(html.content, "lxml")  # 그 내용을 뷰티풀수프 객체로 만들어 수프로 만든다.
       return soup  # 한 번 더 해라.


def title_finder(soup):
   """받은 수프 객체에서 모든 게시물의 링크를 찾아내고 그 개수와 내용을 객체로 리턴한다."""
   link = soup.find_all("tr", {"class": "ub-content"})  # 수프에서 게시물 엘리먼트를 모두 찾아낸다. 크게 잡았다.
   for m in link:
       if m.find("em", {"class": "icon_img icon_notice"}) \
               or m.find("em", {"class": "icon_img icon_issue"}):
           count_trash.append(m.a.text)  # 그 쓸모없는 글의 개수도 카운트를 해둔다.
       elif m.find("em", {"class": "icon_img icon_pic"}) \
               or m.find("em", {"class": "icon_img icon_recomimg"}):  # 이미지를 갖고 있는 게시물이 있으면
           count_title.append(m.a.text)  # 그 게시물들의 제목을 리스트에 추가시켜나간다.
   return link


def inner_link_browser(one_link):
   """리스트 엘리먼트를 받아서 그 주소에 접속해 request 객체를 리턴한다."""
   link = soup.find_all("tr", {"class": "ub-content"})  # 수프에서 게시물 엘리먼트를 모두 찾아낸다. 크게 잡았다.
   for m in link:
       if m.find("em", {"class": "icon_img icon_notice"}) \
               or m.find("em", {"class": "icon_img icon_issue"}):
           count_trash.append(m.a.text)  # 그 쓸모없는 글의 개수도 카운트를 해둔다.
       elif m.find("em", {"class": "icon_img icon_pic"}) \
               or m.find("em", {"class": "icon_img icon_recomimg"}):  # 이미지를 갖고 있는 게시물이 있으면
           count_title.append(m.a.text)  # 그 게시물들의 제목을 리스트에 추가시켜나간다.
   return link

URL = URL_maker("otokonokozuma", 1, 100)
print(URL)
soup = gall_list_parser(URL)
link = title_finder(soup)
print(type(link))

"""딕셔너리로 구현함. 키로 게시물 번호, 값으로 다시 딕셔너리를 넣음.
제목, 내용, 이름, 날짜, 추천 수, 비추천수, 댓글 수.
이거는 따로 파일을 만들어서 보관하도록 함."""

for m in link:
   if m.find("em", {"class": "icon_img icon_notice"}) \
           or m.find("em", {"class": "icon_img icon_issue"}):
       pass  # 공지는 전부 건너뛴다.
   elif m.find("em", {"class": "icon_img icon_pic"}) \
           or m.find("em", {"class": "icon_img icon_recomimg"}):  # 아이콘이 있으면 다음을 실행한다.
       storage_title.append(m.a.text)  #
       storage_url.append(fix_link + m.a.get('href'))

       inner_link = fix_link + m.a.get('href')
       print(inner_link, "+" * 30)

       html = requests.get(inner_link)
       soup = BeautifulSoup(html.content, "lxml")

       link2 = soup.find_all("div", {"class": "appending_file_box"})

       processing = len(storage_title)
       for n in link2:
           storage_inner_title.append(n.li.text)  # 이미지 제목 따왔음.
           storage_inner_url.append(n.li.a.get('href'))  # 이미지 진짜 주소 따왔음.

           nn = str(n)
           inner_soup = BeautifulSoup(nn, "lxml")
           all_link_in_post = inner_soup.find_all("a")

           order = 1
           for a in inner_soup.find_all('a', href=True):
               real_image_link = image_fix_link + a.get('href')
               image_file = requests.get(real_image_link, allow_redirects=False)
               your_name = extention_finder(n.li.text, order)
               open(your_name, 'wb').write(image_file.content)
               order += 1
               #

           print("processing {}/{}".format(processing, len(count_title)))

for i in range(len(storage_title)):
   print("1★", storage_title[i])
   print("2★", storage_url[i])
   print("3★", storage_inner_title[i])
   print("4★", storage_inner_url[i], "\n")

# def picture_downloader():



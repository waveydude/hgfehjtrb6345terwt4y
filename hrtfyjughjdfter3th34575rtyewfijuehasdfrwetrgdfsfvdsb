#!/usr/bin/env python
from bs4 import BeautifulSoup as bs
import requests
import re
import BeautifulSoup
import timeit
import time
#todo implement 2captcha token retrieval, checkout/pay, multithreading, maybe check cart

email=""
password=""
base_url = 'https://www.adidas.com'
product_id = 'B27136'
size = '7.5'

#logging in 
def login():
	print('Initializing Login...')
	headers={
	'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36'
	}
	url="https://cp.adidas.com/web/eCom/en_US/loadsignin?target=account"
	response=session.get(url,headers=headers)

	payload = {
        'username':email,
        'password':password,
        'signinSubmit':'Sign in',
        'IdpAdapterId':'adidasIdP10',
        'SpSessionAuthnAdapterId':'https://cp.adidas.com/web/',
        'PartnerSpId':'sp:demandware',
        'remembermeParam':'',
        'validator_id':'adieComDWus',
        'TargetResource':'https://www.adidas.com/on/demandware.store/Sites-adidas-US-Site/en_US/MyAccount-ResumeLogin?target=account&target=account',
        'InErrorResource':'https://www.adidas.com/on/demandware.store/Sites-adidas-US-Site/en_US/null',
        'loginUrl':'https://cp.adidas.com/web/eCom/en_US/loadsignin',
        'cd':'eCom|en_US|cp.adidas.com|null',
        'app':'eCom',
        'locale':'en_US',
        'domain':'cp.adidas.com',
        'email':'',
        'pfRedirectBaseURL_test':'https://cp.adidas.com',
        'pfStartSSOURL_test':'https://cp.adidas.com/idp/startSSO.ping',
        'resumeURL_test':'',
        'FromFinishRegistraion':'',
        'CSRFToken':'81f57a1f-f105-477e-813b-f955ad398f34'
        }

	url="https://cp.adidas.com/idp/startSSO.ping"
	response=session.post(url,headers=headers,data=payload)

#Parse resume URL
	p = re.compile("resURL = '([a-zA-Z0-9://.]+)'")
	result=re.findall(p,response.text)

	resumeURL=result[-1]

#still logging in..
	response=session.get(resumeURL,headers=headers)

	soup=BeautifulSoup.BeautifulSoup(response.text)

	_action=soup.find('form').get('action')
	_SAMLResponse=soup.find('input',{'name':'SAMLResponse'}).get('value')
	_RelayState=soup.find('input',{'name':'RelayState'}).get('value')

	payload={
	"SAMLResponse":_SAMLResponse,
	"RelayState":_RelayState
	}

#almost logged in...
	url="https://cp.adidas.com"+_action
	response=session.post(url,headers=headers,data=payload)

	soup=BeautifulSoup.BeautifulSoup(response.text)

	_action=soup.find('form').get('action')
	_TargetResource=soup.find('input',{'name':'TargetResource'}).get('value')
	_REF=soup.find('input',{'name':'REF'}).get('value')

	payload={
	"TargetResource":_TargetResource,
	"REF":_REF
	}


#so close...
	url=_action
	response=session.post(url,headers=headers,data=payload)

#Logged in.
	url="https://www.adidas.com/on/demandware.store/Sites-adidas-US-Site/en_US/MyAccount-Show"
	response=session.get(url,headers=headers)

	soup=BeautifulSoup.BeautifulSoup(response.text)

	_accountInfo=soup.find('div',{'class':'accountwelcome'})
	print _accountInfo.find('h1').text

#add to cart function
def add_to_cart():
	headers={
	'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36'
	}

	response = session.get(base_url + '/us/' + product_id + '.html', headers=headers)
	
	url = response.url
	soup = bs(response.text, 'html.parser')

	size_container = soup.find('select', {'name': 'pid'})
	size_val = 'null'
	try:
		for values in size_container.find_all('option'):
			if size == values.string.strip():
				size_val = values['value']
				break
	except:
		print('All sold out!')
		return False, 'null', 'null'
		
	payload = {
		'Quantity': '1',
		'ajax': 'true',
		'layer': 'Add To Bag overlay',
		'masterPid': product_id,
		'pid': size_val
	}


	if size_val != 'null':
		url = base_url + '/on/demandware.store/Sites-adidas-US-Site/en_US/Cart-MiniAddProduct'
		response = session.post(url, data=payload, headers=headers)
		#print response
		if response.status_code == 200:
			print('Shoe was added to cart...')
		# sleep()
		return True, url, size_val
	else:
		print('{} unavailable'.format(size))
		return False, url, size_val

		
# Main thread?
start = timeit.default_timer()
session = requests.Session()
session.headers.update({
	'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) '
	              'Chrome/52.0.2743.116 Safari/537.36',
	'DNT': '1',
	'Accept-Encoding': 'gzip, deflate, sdch',
	'Accept-Language': 'en-US,en;q=0.8,da;q=0.6',
	'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
})

login()
successful, product_url, size_val = add_to_cart()

if successful:

	print(timeit.default_timer()-start)


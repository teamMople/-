# ๐ก๐คฌ Name Server๋ ์ ๋๋ฅผ ํ๋ค๊ฒ ํ๋๊ฐ?
์ ๋ต์ด ์๋์ง ๋ชจ๋ฅด๊ฒ ์ง๋ง ๋์๊ฒ ํ.๋ฒ.๋ ์๊ฐํ  ๊ธฐํ๋ฅผ ์ฃผ์  ์์ง ๋งค๋์ ๋๊ป ๋๋ฌด ๊ฐ์ฌ๋๋ฆฝ๋๋ค๐

## ๋ฌธ์ ์ํฉ 
   ๊ธฐ์กด์ ์กด์ฌํ๋ ํธ์คํธ๋ฅผ ์ญ์ ํ๊ณ  ๋์ผํ ๋๋ฉ์ธ์ผ๋ก ์ ํธ์คํธ๋ฅผ ๋ง๋ค์๋ค.
   ์ด ๊ณผ์ ์์ ์ ํธ์คํธ์ ๋๋ฉ์ธ์ด ์ฐ๊ฒฐ๋์ง ์๋ ์ํฉ์ด ๋ฐ์ํ๊ณ  ๋๋ฅผ ๋๋ฌด๋ ํ๋ค๊ฒํ๋ค...!
   

### ์ด๋ป๊ฒ ํด๊ฒฐ ??
  ์ผ๋จ ping์ ๋์ก๋ค...     
  
  [ec2-user@ip-172-31-11-120 ~]$ ping boiler-plate.org   
  ping: boiler-plate.org: Name or service not known      
    
  DNS๋ค์๊ฒ ์ง์ ํ๋ค๊ฐ ๊ทธ๋ฐ๋์ ๋ค~~~~ ๋ชจ๋ฅธ๋ค๊ณ  ํด์  Name or service not known ์ด ๋์๋ค.
  ์ฌ๊ธฐ์ ์ด๋ฏธ ์ ๋ต์ด ๋์จ๊ฒ ์๋๊น...? ๋ฌผ๋ก  ๊น๋ง๋์ธ ๋๋ ๊ทธ๊ฑธ ๋ชฐ๋๋ค ใใ     
  ์ฐจ๊ทผ์ฐจ๊ทผ ์ดํด๋ณด์   ๋จผ์  Whois๋ก ์ด ๋๋ฉ์ธ์ด ๋ด๊ฐ ๋ฑ๋กํ ๋๋ฉ์ธ ์ ๋ณด์ ์ผ์นํ๋์ง ํ์ธํ์(whois๋ ๋๋ฉ์ธ์ ์ฃผ์ธ์ด ๋๊ตฌ์ธ์ง ์๋ ค์ฃผ๋ ๋ช๋ น์ด ์ด๋ค)   
  ![image](https://user-images.githubusercontent.com/67067346/161342150-0927504c-b3a0-4496-979c-10bd3e8cfa42.png)   
  ๋๊ทธ๋ผ๋ฏธ์น ๋ถ๋ถ์ด ์ด์ํ๋ค... ๋ด๊ฐ ํธ์คํํ ๋๋ฉ์ธ์๋    
  ns-1553.awsdns-02.co.uk.   
  ns-184.awsdns-23.com.   
  ns-1063.awsdns-04.org.   
  ns-861.awsdns-43.net.     
  ์ด๋ ๊ฒ ์์ฑ์ด ๋์ด์์๋ค. ์ด NS๋ฅผ ์ผ์น์ํค๋ฉด ๋ฌธ์ ๋ ๋์ด๋ค~(ROUTE53์ ๊ฒฝ์ฐ ํธ์คํ์์ญ -> ์ธ๋ถ์ ๋ณด์์ ์์ ํ์!)
  
### ์ ๋ฐ์ํ ๊ฑฐ์ผ??
  ๊ธฐ์กด์ ๋ฑ๋กํ ํธ์คํ ์์ญ์ด ์์ง ์์ ๋์ง ์์๊ธฐ ๋๋ฌธ์ด๋ค.
  ์ ํํ๋ AWS์์ ์์ ๋์ง ์์ ๊ฒ ๊ฐ๋ค. ๊ทธ๋์ ์ด ์์์ ์๋์ผ๋ก ์งํํด์ ๋ฐ๋ NS๋ก ์ธ์ํ๊ฒ ํด์ฃผ๋ ์์์ด ํ์ํ๋ ๊ฒ.
  (์๊ฐ์ด ์ง๋๋ฉด ์๋์ผ๋ก ๋ฐ๊ฟ์ฃผ๋โฆ?)

  


  
  
  
  

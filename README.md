insert-update-image
===================




import random
import glob
import multiprocessing
import time 
import MySQLdb
import ast
from datetime import datetime
import sys
num_fetch_threadsm = 5
enclosure_queuem = multiprocessing.JoinableQueue()
import sys
import logging

logging.basicConfig(level=logging.DEBUG, format='[%(levelname)s] (%(threadName)-10s) %(message)s', )
def my_strip(x):
    try:
        x = str(x).strip()
        x = MySQLdb.escape_string(x).strip()
    except:
        x = str(x.encode("ascii", "ignore")).strip()
        x = MySQLdb.escape_string(x).strip()

    return x

        


def main(link):
            db = MySQLdb.connect("localhost","root","6Tresxcvbhy")
            cursor = db.cursor()
            sql =""" use zivame """
            cursor.execute(sql)

                      
       # try:
            line_split = ast.literal_eval(str(link).strip())
            line_split =tuple(map(my_strip, line_split))

         
            #product_id = None

            sql_select = """ select product_id from `zivame_images` where `product_id` = "%s" """ %(str(line_split[0]).strip())
            cursor.execute(sql_select)
            results = cursor.fetchone()

           
            sqlentry = """insert into zivame_images (product_id,image_link )  values ("%s", "%s")"""

            #print "hello"
            sqlentry = sqlentry %(line_split) 
            logging.debug("inserted..................................................")
         
                

            try:
                cursor.execute(sqlentry)
                db.commit()

            except:
                try:
                    db.ping()
                    cursor.execute(sqlentry)
                    db.commit()
                    print "connection reconnect..................................................................................."
                    print result
                except:
                    db.rollback()
                    print "rollback. .............................................................................................."
        
      #  except:
       #     pass    
            
            db.close()
            



def main3(i, q):
    for link in iter(q.get, None):
        try:
            main(link)
        except:
            pass

        time.sleep(2)
        q.task_done()
              
    q.task_done()



def supermain():
    file_name ="/home/desktop/anit/zivame/image_files.csv"
    #print file_name
    f =open(file_name)
    procs = []

    for i in range(num_fetch_threadsm):
        procs.append(multiprocessing.Process(name = str(i), target=main3, args=(i, enclosure_queuem,)))
        procs[-1].start()

    for link in f:
        #print link
        enclosure_queuem.put(link.strip())

    enclosure_queuem.join()

    for p in procs:
        enclosure_queuem.put(None)

    enclosure_queuem.join()

    for p in procs:
        p.join(25)    


    db.close()

    


if __name__=="__main__":
    tstart = datetime.now()
   # directory = "/home/desktop/working_sites/flipkart_site/flipkart/flipkart_dir190614"
    file_name ="image_files.csv"
    #directory ="cilory_dir"
    supermain()
    #line = "['VCB-02', 'http://cdn.zivame.com/media/catalog/product/cache/1/image/410x/040ec09b1e35df139433887a97daa66f/c/a/cake_vanilla_cream_bikini_brief.jpg']"
    #main(line)
    tend = datetime.now()
    print tend - tstart

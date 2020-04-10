---
title: Filesystem benchmarking
author: Andrew Ludwar
type: post
date: 2016-09-12T02:51:38+00:00
url: /blog/2016/09/11/filesystem-benchmarking/
dsq_thread_id:
  - 5825498242
categories:
  - open source
  - performance tuning
  - storage
tags:
  - I/O
  - performance tuning
  - SSD
  - storage

---
Once in a while I need to do some I/O benchmarking. Either when creating a baseline for a before and after performance tuning comparison, or just because I want to see how fast my new SSD/M.2 really is. The tool I find myself going to over the years has been [iozone][1]. I like it because it easily does a myriad of tests that I don&#8217;t need to create or think about how to simulate myself, and it&#8217;s easily installed on any linux distro. It also outputs the test data into a format that is easily imported into a spreadsheet. I recently added a [couple SSD][2]s to my workstation, and finally got around to benchmarking them.

IOzone has a slew of pre-built tests you can run, but I typically just do 5 (read, random read, write, random write, and stride read). From the iozone help:

<pre class="">          -i #  Test to run (0=write/rewrite, 1=read/re-read, 2=random-read/write
                 3=Read-backwards, 4=Re-write-record, 5=stride-read, 6=fwrite/re-fwrite
                 7=fread/Re-fread, 8=random_mix, 9=pwrite/Re-pwrite, 10=pread/Re-pread
                 11=pwritev/Re-pwritev, 12=preadv/Re-preadv)</pre>

Within each test you can also specify the record size, and file size to be written. This is my usual test syntax:

<pre class="">$ /opt/iozone/bin/iozone  -a -b benchmark.xls -R -i 0 -i 1 -i 2 -i 5 -i 8 -f /export/testfile
    Iozone: Performance Test of File I/O
            Version $Revision: 3.452 $
        Compiled for 32 bit mode.
        Build: linux 

    Contributors:William Norcott, Don Capps, Isom Crawford, Kirby Collins
                 Al Slater, Scott Rhine, Mike Wisner, Ken Goss
                 Steve Landherr, Brad Smith, Mark Kelly, Dr. Alain CYR,
                 Randy Dunlap, Mark Montague, Dan Million, Gavin Brebner,
                 Jean-Marc Zucconi, Jeff Blomberg, Benny Halevy, Dave Boone,
                 Erik Habbinga, Kris Strecker, Walter Wong, Joshua Root,
                 Fabrice Bacchella, Zhenghua Xue, Qin Li, Darren Sawyer,
                 Vangel Bojaxhi, Ben England, Vikentsi Lapa,
                 Alexey Skidanov.

    Run began: Sun Sep 11 20:17:18 2016

    Auto Mode
    Excel chart generation enabled
    Command line used: /opt/iozone/bin/iozone -a -b benchmark.xls -R -i 0 -i 1 -i 2 -i 5 -i 8 -f /export/testfile
    Output is in kBytes/sec
    Time Resolution = 0.000001 seconds.
    Processor cache size set to 1024 kBytes.
    Processor cache line size set to 32 bytes.
    File stride size set to 17 * record size.
                                                              random    random     bkwd    record    stride                                    
              kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
              64       4   888413   953673  4297180  3033971  2786676  1120480                      2463305                                   
              64       8   744494  1232721  3551002  4261142  3047037  1422167                      2553108                                   
              64      16   864466  1560183  4958872  5328791  3356712  1778661                      2277099                                   
              64      32  1182756  1335389  8051483  6329941  7985638  1783120                      2914882                                   
              64      64  1453111  1598043  7075540 12653171  4552275  1887599                      2207552                                   
             128       4   999800  1195499  3375208  4131854  3648013  1855621                      2783700                                   
             128       8  1164348  1487891  4404899  5328896  3660474  1706333                      5548323                                   
             128      16  1348377  1704441  4738496  4003638  5355501  2459229                      3883374                                   
             128      32  1437115  1704867  5318585  4721384  5144080  2243177                      4142611                                   
             128      64  1049847  1523170  7979556  6401923  4269068  2067294                      4409651                                   
..</pre>

&#8230; and so on as it goes through all of the test scenarios. At the end, I&#8217;m given a nicely formatted spreadsheet that&#8217;s easy to make graphs from:

[<img class="size-large wp-image-435" src="https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example-1024x867.png" alt="iozone graph output" width="1024" height="867" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example-1024x867.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example-300x254.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example-768x650.png 768w, https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example.png 1285w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

Looking at my read stats, at the peak, I&#8217;m getting just over 20 GB/sec. However, the average seems to be about 12-14 GB/sec. Not too shabby! (I&#8217;ve got a few SSDs in a RAID1 array). Full test details are in a [spreadsheet][4] on github.

&nbsp;

 [1]: http://iozone.org/
 [2]: http://www.newegg.ca/Product/Product.aspx?Item=N82E16820147373
 [3]: https://calgaryrhce.ca/wp-content/uploads/2016/09/iozone-example.png
 [4]: https://github.com/aludwar/scripts/blob/master/iozone/workstation.odc
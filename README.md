# Web Content Crawler
&nbsp;

## Design
The Web Content Crawler will consist of 4 major compoments
1. #### **Controller**
* The controller will be implemented as a gen_server 
* Controller will maintian a state, which include: 
    * `url_status` <br/>
      A map of `%{url=>status}` <br/>
      The status can be `nil`, `:requested`, `:processed` <br/>
      A URL, if found in this map and the status is not `nil`,  will be rejected 
    * `urls_to_process` <br/>
      The queue of the URLs that should be crawled. <br/>
      An idle crawler will ask controller dequeue and send over a URL. 
    * `crawlers` <br/>
      A mapset of `crawler`s
    * A queue of available `saver`s <br/>
      An idle `saver` will ask `controller` to enque itself <br/>
      A crawler will ask 'controller' to deque a `saver` and it will send the HTTP response to the saver.
* The contoller can check the ratio of "processed" url and control the pace of crawler
&nbsp;
2. #### **Crawler**
* Crawler will be implemented as a gen_server. 
* Crawler can be distributed. Mutiple nodes, multiple Crawlers. 
* Crawler will do the following things:
    * Register itself to the controller
    * React to controller's request to `idle` or `run`
    * Request controler to dispatch a URL
    * Send the HTTP request with that URL
    * Tell controller a HTTP request has been sent for the URL
    * Get the content out of the HTTP response and do:
        * ask controller to dequeue an availabe saver and send the content to that saver
        * send the controller all the URL links in that content
&nbsp;
3. #### **Saver**
* Saver will be implemented as a gen_server. 
* Multiple savers can run. 
* Saver will do:
    * register itself to the Controller
    * receive the content of a URL and save it in the file system, and request the controller to enqueu itself.
    * tell the controller the content of the URL is saved
&nbsp;
4. #### **Console**
* A console taking standard input and ask the controller's action
    * start with a URL
    * stop
    * change `pace_ratio` -- the percentage of `:processed` URLs
    * show the `url_status` map
    * etc etc ...
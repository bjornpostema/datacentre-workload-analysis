# Data Analysis of the Search Queries Data Set
This document regards the data analysis of a data centre containing 10 nodes, which main job is to respond to search queries send to these nodes send by customers via the Internet. The data range from 10th of August, 2016 to 31th of August, 2016; and from 26th of September, 2016 to 17th of October, 2016. The total set contains about 7.6 Million/19.6 Million entries/rows and its size is unpacked about 550 MB/1401 MB in comma seperated CSV format and packed about 71 MB/211 MB in GZIP format. The data set contains information about date & time, service node identifiers (cluster_host), request method, response size, status code, duration and processing duration of each job.

The main purpose of this document is to discover workload in a meanful way, that will serve as an input for the *data centre simulation framework* (DCSF) in AnyLogic. DCSF is capable of handling many styles of input. This meanful way would be more valuable if it is reproducable. Therefore, Jupyter Notebook is used to document and write code. *Jupyter Notebook* is used together with *Python* that import the libraries *Pandas* for data science, *NumPy* for numerical math and *Matplotlib* for plotting purposes.

The data set used in this analysis is external and not accessible. Yet, the most valuable output and code for reproducability is displayed below. The data set is stored in a Dutch storage company called *Stack*, who claims to have high security and to be independant from unwanted external observers. The server of Jupyter Notebook is currently situated at University of Twente.

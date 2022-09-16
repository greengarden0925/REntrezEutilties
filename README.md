# REntrezEutilties
Retrieve NCBI database (GEO) searched data summarries using Entrez e-utilties api via R code  


The following is the  R script!
###########################################

install.packages("XML")
install.packages(c("httr", "jsonlite"))
library(httr)
library(jsonlite)
library(XML)
library(rvest)
library(dplyr)
library(xml2 )

##用Entrez esearch api去抓查詢資料紀錄的UIDs (該筆資料在Entrez database的ID)
#esearch.fcgi?db=database&term=query
#查詢GEO想要的資料條件其文件ID

q1="https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=gds&term=Homo+sapiens[orgn]+AND+expression+profiling+by+high+throughput+sequencing[DataSet+Type]+AND+colorectal+cancer[MeSH+Terms]+AND+recurrence[MeSH+Terms]&retmax=50&usehistory=y"
r1 = GET(q1)


xml.doc <- xmlParse(r1)
xml.top <- xmlRoot(xml.doc) 
xml.top
ids=xmlToDataFrame(xml.top[[6]])[[1]] #取得UIDs
ids

#用Entrez esummary api去抓資料紀錄摘要
##esummary.fcgi?db=database&id=uid1,uid2,uid3,...

q2=sprintf("https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=gds&id=%s",
           paste0(ids,collapse=","))
q2

res = GET(q2)
res

#並解析XML資料
xml.doc <- xmlParse(res)
xml.doc

# 取出 XML 的根節點
xml.top <- xmlRoot(xml.doc)
xml.top

# 查看節點名稱
xmlName(xml.top)

# 查看子節點數量
xmlSize(xml.top)

# 查看子結點
names(xml.top)



#測試取第一筆資料摘要並轉成data.frame格式
xml.df <- xmlToDataFrame(xml.top[1])
xml.df

#用迴圈取出26筆資料摘要
dts=NULL
for(i in 1:xmlSize(xml.top)){
  xml.df <- xmlToDataFrame(xml.top[i])
  dts=rbind(dts,xml.df)
  
}

dts[1:10,1:5]

library(openxlsx)
write.xlsx(dts,file="D:\\query.xlsx",rowNames=F,colNames=F)



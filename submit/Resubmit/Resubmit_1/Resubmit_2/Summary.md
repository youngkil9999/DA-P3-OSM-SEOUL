<!DOCTYPE html><html><head><meta charset="utf-8"><title>Untitled Document.md</title><style></style></head><body id="preview">
<h1><a id="OpenStreetMap_Data_Case_Study_1"></a>OpenStreetMap Data Case Study</h1>
<h3><a id="1_Map_Area_3"></a>1. Map Area</h3>
<ul>
<li>Seoul Metro area</li>
<li>Source : <a href="https://s3.amazonaws.com/metro-extracts.mapzen.com/seoul_south-korea.osm.bz2">https://s3.amazonaws.com/metro-extracts.mapzen.com/seoul_south-korea.osm.bz2</a></li>
</ul>
<h3><a id="2_Problems_Encountered_in_A_Map_7"></a>2. Problems Encountered in A Map</h3>
<h5><a id="21_Needed_to_modigy_ro_and_gil_8"></a>2-1. Needed to modigy ‘ro’ and ‘gil’</h5>
<ul>
<li>‘v’ value in OSM data</li>
<li>‘v’ value = xxxgil or xxxro ( which is the street name )</li>
</ul>
<pre><code class="language-sh">    &lt;way id=<span class="hljs-string">"59079489"</span> version=<span class="hljs-string">"2"</span> timestamp=<span class="hljs-string">"2015-06-21T12:19:44Z"</span> changeset=<span class="hljs-string">"32115786"</span> uid=<span class="hljs-string">"1696339"</span> user=<span class="hljs-string">"topstone"</span>&gt;
        &lt;tag k=<span class="hljs-string">"name"</span> v=<span class="hljs-string">"언주로 (Eonjuro)"</span>/&gt;
        &lt;tag k=<span class="hljs-string">"name:ko"</span> v=<span class="hljs-string">"언주로"</span>/&gt;
        &lt;tag k=<span class="hljs-string">"name:ko_rm"</span> v=<span class="hljs-string">"Eonjuro"</span>/&gt;
    &lt;way id=<span class="hljs-string">"59174373"</span> version=<span class="hljs-string">"8"</span> timestamp=<span class="hljs-string">"2015-10-23T11:12:07Z"</span> changeset=<span class="hljs-string">"34818482"</span> uid=<span class="hljs-string">"1385794"</span> user=<span class="hljs-string">"hyolee2"</span>&gt;
        &lt;tag k=<span class="hljs-string">"name"</span> v=<span class="hljs-string">"탄천길"</span>/&gt;
        &lt;tag k=<span class="hljs-string">"ncat"</span> v=<span class="hljs-string">"광역시도로"</span>/&gt;
        &lt;tag k=<span class="hljs-string">"name:ko"</span> v=<span class="hljs-string">"탄천길"</span>/&gt;
        &lt;tag k=<span class="hljs-string">"name:ko_rm"</span> v=<span class="hljs-string">"Tancheongil"</span>/&gt;
</code></pre>
<ul>
<li>but actually the street name normally xxx - gil and xxx - ro</li>
<li>So, Street name changed, gil &gt;&gt; -gil, ro &gt;&gt; -ro</li>
<li>for example, normally used Wall-street, 34th-street in KOREA, but it was typed Wallstreet, 34thstreet in OSM data</li>
</ul>
<h5><a id="22_Needed_to_remove_korean_characters_and_just_leave_english_characters_inside_parenthesis_26"></a>2-2. Needed to remove korean characters and just leave english characters inside parenthesis</h5>
<ul>
<li>v values used korean and english together</li>
<li>difficult to recognize</li>
<li>v = ‘Korean’ ( ‘English’ )</li>
</ul>
<pre><code class="language-sh">    &lt;node id=<span class="hljs-string">"368904647"</span> lat=<span class="hljs-string">"37.480399"</span> lon=<span class="hljs-string">"126.930559"</span> version=<span class="hljs-string">"2"</span> timestamp=<span class="hljs-string">"2012-10-27T05:53:18Z"</span> changeset=<span class="hljs-string">"13645644"</span> uid=<span class="hljs-string">"371753"</span> user=<span class="hljs-string">"sanha"</span>&gt;
            &lt;tag k=<span class="hljs-string">"name"</span> v=<span class="hljs-string">"신관주유소 (Singwan Gas Station)"</span>/&gt;
            &lt;tag k=<span class="hljs-string">"ncat"</span> v=<span class="hljs-string">"주유소"</span>/&gt;
</code></pre>
<ul>
<li>remove korean just leave english</li>
<li>only leave english. make it easy to see such like this.  v : English</li>
</ul>
<h3><a id="3_Modify_V_value_ro_and_gil_and_also_Modify_both_Korean_and_English_on_V_value_38"></a>3. Modify V value ‘ro’ and ‘gil’ and also Modify both Korean and English on V value</h3>
<p>This function will modify all the characters into the form desired which addressed above, delete korean characters and
leave only english inside parenthesis. also, it will also modify the characters ‘ro’ and ‘gil’ to ‘-ro’ and ‘-gil’</p>
<pre><code>mapping = {&quot;gil&quot;: &quot;-gil&quot;, &quot;ro&quot;: &quot;-ro&quot;}
expected = [&quot;-gil&quot;, &quot;-ro&quot;]
word = []
words = []

def replace_value(name, mapp):
    if re.search('-gil', name):
        pass
    else :
        if re.search('gil', name):
            name  = re.sub('gil', '-gil', name)
    koen = KorToEng(name)
    return koen

def KorToEng(data):
    if len(re.split(r'[()]',data)) &gt; 1:
        word = len(re.split(r'[()]',data))
        words = re.split(r'[()]', data)
        if word == 3:
            data = words[1]
    else:
        pass
    return data
</code></pre>
<h3><a id="4_DATA_OVERVIEW_67"></a>4. DATA OVERVIEW</h3>
<p>File Size</p>
<p>only the reason sqlite data bigger than the original osm data is because I made new tables using each node and way table</p>
<pre><code>seoul_south-korea.osm ........284.1MB
new.db .......................319MB
node_group.csv ...............107.5MB
node_tag_group.csv............11.3MB
way_group.csv ................8.7MB
way_tag_group.csv ............9.3MB
</code></pre>
<h3><a id="41_OSM_Data_Analysis_79"></a>4-1. OSM Data Analysis</h3>
<ul>
<li>Here is OSM data root tree</li>
<li>I chose to analyze ‘node’ and ‘way’ tag in SQL</li>
</ul>
<p>Node, Way Counts</p>
<pre><code>sqlite&gt; select count(*) from userTable;
sqlite&gt; select count(*) from newTable;
</code></pre>
<p>1443573,
640767</p>
<h5><a id="411_Unique_User_ID_93"></a>4-1-1 Unique User ID</h5>
<pre><code class="language-sh">sqlite&gt; select user, count(*) as num from userTable
   ...&gt; group by user
   ...&gt; having num = <span class="hljs-number">1</span>;
</code></pre>
<pre><code class="language-sh"><span class="hljs-number">2</span>hertz|<span class="hljs-number">1</span>
<span class="hljs-number">8</span>iseoii|<span class="hljs-number">1</span>
Agmenor|<span class="hljs-number">1</span>
Amigos_Conmigo|<span class="hljs-number">1</span>
Ashs1126|<span class="hljs-number">1</span>
BBIBBIBi|<span class="hljs-number">1</span>
BG2YF|<span class="hljs-number">1</span>
BarrettHC|<span class="hljs-number">1</span>
Boppet|<span class="hljs-number">1</span>
BugBuster|<span class="hljs-number">1</span>
CG Kang|<span class="hljs-number">1</span>
CeteraTolle|<span class="hljs-number">1</span>
Christian Westhoff|<span class="hljs-number">1</span>
Chunsoo Lee|<span class="hljs-number">1</span>
CloCkWeRX|<span class="hljs-number">1</span>
DHKIM|<span class="hljs-number">1</span>
Dahee Choe|<span class="hljs-number">1</span>
Deen|<span class="hljs-number">1</span>
Dio Seo|<span class="hljs-number">1</span>
Dongyi|<span class="hljs-number">1</span>
Edatmpa|<span class="hljs-number">1</span>
Enzin333|<span class="hljs-number">1</span>
Ethan Lee|<span class="hljs-number">1</span>
Fallenstar|<span class="hljs-number">1</span>
Fumiko|<span class="hljs-number">1</span>
Gamayun|<span class="hljs-number">1</span>
GarionPark|<span class="hljs-number">1</span>
GercoKees|<span class="hljs-number">1</span>
Gilbert C|<span class="hljs-number">1</span>
Gyeongmoon Eom|<span class="hljs-number">1</span>
HHCacher|<span class="hljs-number">1</span>
Hankie|<span class="hljs-number">1</span>
Hero Kim|<span class="hljs-number">1</span>
Hojin Shin|<span class="hljs-number">1</span>
HoloDuke|<span class="hljs-number">1</span>
HorribleChoice|<span class="hljs-number">1</span>
Hue Lee|<span class="hljs-number">1</span>
HwangJ<span class="hljs-keyword">in</span>Woo|<span class="hljs-number">1</span>
Ikjune Yoon|<span class="hljs-number">1</span>
JISU|<span class="hljs-number">1</span>
JK_KOREA|<span class="hljs-number">1</span>
JS Kim|<span class="hljs-number">1</span>
Jae Sung Han|<span class="hljs-number">1</span>
Jang_ys|<span class="hljs-number">1</span>
Jean Kim|<span class="hljs-number">1</span>
Jedrzej Pelka|<span class="hljs-number">1</span>
Jewon|<span class="hljs-number">1</span>
Jochen Topf|<span class="hljs-number">1</span>
June Rainberry|<span class="hljs-number">1</span>
Junios|<span class="hljs-number">1</span>
Junmo Kim|<span class="hljs-number">1</span>
K83|<span class="hljs-number">1</span>
Kang Seung sik|<span class="hljs-number">1</span>
Kimbabchapati|<span class="hljs-number">1</span>
Lee Sehoon|<span class="hljs-number">1</span>
Lee jae hyung|<span class="hljs-number">1</span>
Louis JANG|<span class="hljs-number">1</span>
Luis36995_labuildings|<span class="hljs-number">1</span>
M31sa|<span class="hljs-number">1</span>
MAPconcierge|<span class="hljs-number">1</span>
Math1985|<span class="hljs-number">1</span>
Matthewmayer|<span class="hljs-number">1</span>
Maverick LEE|<span class="hljs-number">1</span>
M<span class="hljs-keyword">in</span>Seok|<span class="hljs-number">1</span>
NTUKLC|<span class="hljs-number">1</span>
Nam Wook Kim|<span class="hljs-number">1</span>
None <span class="hljs-number">1</span>|<span class="hljs-number">1</span>
Nuk34|<span class="hljs-number">1</span>
Oberpfälzer|<span class="hljs-number">1</span>
Orca0418|<span class="hljs-number">1</span>
Pelcos|<span class="hljs-number">1</span>
Philogrammer|<span class="hljs-number">1</span>
Phrast|<span class="hljs-number">1</span>
RailMapper|<span class="hljs-number">1</span>
Rainbow International School|<span class="hljs-number">1</span>
RehobothGlobal|<span class="hljs-number">1</span>
Rodrigo Rega|<span class="hljs-number">1</span>
Rouni|<span class="hljs-number">1</span>
SGPark|<span class="hljs-number">1</span>
SNJeong|<span class="hljs-number">1</span>
S_young|<span class="hljs-number">1</span>
Sandi Khim|<span class="hljs-number">1</span>
Sangsun Park|<span class="hljs-number">1</span>
Sean Lee|<span class="hljs-number">1</span>
Shane Metzger|<span class="hljs-number">1</span>
SomeRandomMe|<span class="hljs-number">1</span>
StellarKoala|<span class="hljs-number">1</span>
Sunny Park|<span class="hljs-number">1</span>
TJ Lee|<span class="hljs-number">1</span>
TJJU|<span class="hljs-number">1</span>
Ted Kwon|<span class="hljs-number">1</span>
Tribunal Supremo Electoral|<span class="hljs-number">1</span>
U20|<span class="hljs-number">1</span>
VisualDive|<span class="hljs-number">1</span>
VlIvYur|<span class="hljs-number">1</span>
Wikidata|<span class="hljs-number">1</span>
Will3301|<span class="hljs-number">1</span>
YCH|<span class="hljs-number">1</span>
Yong-Chan Kim|<span class="hljs-number">1</span>
ZeboStoneleigh|<span class="hljs-number">1</span>
ajashton|<span class="hljs-number">1</span>
angys2|<span class="hljs-number">1</span>
arthur8|<span class="hljs-number">1</span>
asdf2|<span class="hljs-number">1</span>
awesomekimbos|<span class="hljs-number">1</span>
bes_internal|<span class="hljs-number">1</span>
bioshome|<span class="hljs-number">1</span>
chorani81|<span class="hljs-number">1</span>
chris johnston|<span class="hljs-number">1</span>
coolx|<span class="hljs-number">1</span>
cricket|<span class="hljs-number">1</span>
crucialsoft2012|<span class="hljs-number">1</span>
dalee02|<span class="hljs-number">1</span>
davidjudekim|<span class="hljs-number">1</span>
dbdbdpdp72|<span class="hljs-number">1</span>
dcs|<span class="hljs-number">1</span>
devmario|<span class="hljs-number">1</span>
disBtopia|<span class="hljs-number">1</span>
dk_bm|<span class="hljs-number">1</span>
dp2743gw|<span class="hljs-number">1</span>
dua804|<span class="hljs-number">1</span>
elucian|<span class="hljs-number">1</span>
emersion|<span class="hljs-number">1</span>
engiYong|<span class="hljs-number">1</span>
etajin|<span class="hljs-number">1</span>
falcon31|<span class="hljs-number">1</span>
gabyul|<span class="hljs-number">1</span>
geruto|<span class="hljs-number">1</span>
gsa|<span class="hljs-number">1</span>
gtggtg|<span class="hljs-number">1</span>
hanlong|<span class="hljs-number">1</span>
hansumo81|<span class="hljs-number">1</span>
hater|<span class="hljs-number">1</span>
heavens|<span class="hljs-number">1</span>
hgfonseca|<span class="hljs-number">1</span>
hkucharek|<span class="hljs-number">1</span>
hyeri jung|<span class="hljs-number">1</span>
hynho|<span class="hljs-number">1</span>
iamphoto|<span class="hljs-number">1</span>
idasinc|<span class="hljs-number">1</span>
ideacactus|<span class="hljs-number">1</span>
ireneNK|<span class="hljs-number">1</span>
itisjune|<span class="hljs-number">1</span>
iymlmsh|<span class="hljs-number">1</span>
jc67|<span class="hljs-number">1</span>
jdinjdi|<span class="hljs-number">1</span>
jee1033|<span class="hljs-number">1</span>
jengelh|<span class="hljs-number">1</span>
jeonseik|<span class="hljs-number">1</span>
jhparker|<span class="hljs-number">1</span>
jinalfoflia|<span class="hljs-number">1</span>
jjspride|<span class="hljs-number">1</span>
jong b|<span class="hljs-number">1</span>
jongho84|<span class="hljs-number">1</span>
jongseon son|<span class="hljs-number">1</span>
joyamor|<span class="hljs-number">1</span>
kangdaejang|<span class="hljs-number">1</span>
kjh1592|<span class="hljs-number">1</span>
kortinu|<span class="hljs-number">1</span>
labsurde|<span class="hljs-number">1</span>
lee kn|<span class="hljs-number">1</span>
leeSS|<span class="hljs-number">1</span>
leejeongjun|<span class="hljs-number">1</span>
legotown|<span class="hljs-number">1</span>
mailiam|<span class="hljs-number">1</span>
mapfreak|<span class="hljs-number">1</span>
mbncg|<span class="hljs-number">1</span>
mhu283|<span class="hljs-number">1</span>
mihaii_telenav|<span class="hljs-number">1</span>
miurahr|<span class="hljs-number">1</span>
moningbel|<span class="hljs-number">1</span>
monsterlee48|<span class="hljs-number">1</span>
moren|<span class="hljs-number">1</span>
moyogo|<span class="hljs-number">1</span>
mozolondon|<span class="hljs-number">1</span>
mtmail|<span class="hljs-number">1</span>
mubyoung|<span class="hljs-number">1</span>
myforest|<span class="hljs-number">1</span>
myjoey|<span class="hljs-number">1</span>
pierre1011|<span class="hljs-number">1</span>
popcornhouse|<span class="hljs-number">1</span>
ptateosi|<span class="hljs-number">1</span>
qordytpq|<span class="hljs-number">1</span>
raiseone|<span class="hljs-number">1</span>
raymondnewby|<span class="hljs-number">1</span>
redsnow78|<span class="hljs-number">1</span>
robleiphart|<span class="hljs-number">1</span>
rsnaru|<span class="hljs-number">1</span>
samiupsala|<span class="hljs-number">1</span>
sangukim|<span class="hljs-number">1</span>
sarojaba|<span class="hljs-number">1</span>
schumyHH|<span class="hljs-number">1</span>
seiwonKim|<span class="hljs-number">1</span>
selena0803|<span class="hljs-number">1</span>
seogain|<span class="hljs-number">1</span>
sh221b|<span class="hljs-number">1</span>
shiftkey|<span class="hljs-number">1</span>
sk92129|<span class="hljs-number">1</span>
smitchell78|<span class="hljs-number">1</span>
taijin|<span class="hljs-number">1</span>
tcochr|<span class="hljs-number">1</span>
techsavvy|<span class="hljs-number">1</span>
temip|<span class="hljs-number">1</span>
tillluna|<span class="hljs-number">1</span>
tlacoyo|<span class="hljs-number">1</span>
toshilow|<span class="hljs-number">1</span>
triumph47|<span class="hljs-number">1</span>
ukaszyk|<span class="hljs-number">1</span>
us38914|<span class="hljs-number">1</span>
user_5359|<span class="hljs-number">1</span>
vectorking|<span class="hljs-number">1</span>
voland|<span class="hljs-number">1</span>
wallasoft|<span class="hljs-number">1</span>
widelake|<span class="hljs-number">1</span>
wieland|<span class="hljs-number">1</span>
won21kr|<span class="hljs-number">1</span>
woo seungwan|<span class="hljs-number">1</span>
wuchoi|<span class="hljs-number">1</span>
yesbento|<span class="hljs-number">1</span>
yiqingj|<span class="hljs-number">1</span>
ypid|<span class="hljs-number">1</span>
yurasi|<span class="hljs-number">1</span>
zcom|<span class="hljs-number">1</span>
zorque|<span class="hljs-number">1</span>
Дмитрий Александров|<span class="hljs-number">1</span>
عقبة بن نافع|<span class="hljs-number">1</span>
陳信翰|<span class="hljs-number">1</span>
가면술사|<span class="hljs-number">1</span>
거친마루|<span class="hljs-number">1</span>
구정모|<span class="hljs-number">1</span>
권중규|<span class="hljs-number">1</span>
귀신시바|<span class="hljs-number">1</span>
김석용|<span class="hljs-number">1</span>
김설기|<span class="hljs-number">1</span>
김송권|<span class="hljs-number">1</span>
김형태|<span class="hljs-number">1</span>
류재관|<span class="hljs-number">1</span>
박종태|<span class="hljs-number">1</span>
산티니 전국대리점 현황|<span class="hljs-number">1</span>
서울대GIS실습계정<span class="hljs-number">002</span>|<span class="hljs-number">1</span>
서울대공간정보교육<span class="hljs-number">001</span>|<span class="hljs-number">1</span>
설레임|<span class="hljs-number">1</span>
성안교회|<span class="hljs-number">1</span>
양갱좋앙|<span class="hljs-number">1</span>
어디서본아저씨|<span class="hljs-number">1</span>
얼린사과|<span class="hljs-number">1</span>
예림예|<span class="hljs-number">1</span>
온천거북|<span class="hljs-number">1</span>
우린굉장해|<span class="hljs-number">1</span>
윤오리|<span class="hljs-number">1</span>
의정부침례교회|<span class="hljs-number">1</span>
이의재|<span class="hljs-number">1</span>
정성태|<span class="hljs-number">1</span>
최종철|<span class="hljs-number">1</span>
카리브레스토랑|<span class="hljs-number">1</span>
토토로검도|<span class="hljs-number">1</span>
푸른하늘|<span class="hljs-number">1</span>
하늘여행|<span class="hljs-number">1</span>
한양내과|<span class="hljs-number">1</span>
</code></pre>
<h3><a id="5_The_most_frequently_presented_key_on_SEOUL_map_362"></a>5. The most frequently presented key on SEOUL map</h3>
<pre><code>sqlite&gt; select key, count(*) as num from newTable
   ...&gt; group by key
   ...&gt; order by num desc
   ...&gt; limit 30;
</code></pre>
<p>Here the top ten results</p>
<pre><code class="language-sh">highway|<span class="hljs-number">102643</span>
name|<span class="hljs-number">85685</span>
name:en|<span class="hljs-number">57537</span>
<span class="hljs-built_in">source</span>|<span class="hljs-number">46036</span>
name:ko_rm|<span class="hljs-number">44405</span>
name:ko|<span class="hljs-number">43892</span>
ncat|<span class="hljs-number">36757</span>
building|<span class="hljs-number">32725</span>
amenity|<span class="hljs-number">23033</span>
oneway|<span class="hljs-number">15483</span>
</code></pre>
<h2><a id="6_Additional_Data_Exploration_380"></a>6. Additional Data Exploration</h2>
<h3><a id="61_The_most_frequently_presented_amenity_on_SEOUL_map_382"></a>6-1. The most frequently presented amenity on SEOUL map</h3>
<pre><code>sqlite&gt; select newTable.value, count(*) as num from newTable
   ...&gt; where newTable.key = 'amenity'
   ...&gt; group by newTable.value
   ...&gt; order by num desc;
</code></pre>
<p>Here is the top five results</p>
<pre><code class="language-sh">hospital|<span class="hljs-number">5199</span>
school|<span class="hljs-number">3167</span>
dentist|<span class="hljs-number">1691</span>
restaurant|<span class="hljs-number">1428</span>
bank|<span class="hljs-number">1366</span>
</code></pre>
<p>It looks reasonable that a hospital is on a first place. This is actually true that korean is the best for the quality of the hospital based on the cost because the goverment control the health insurance system not dominated by the private insurance company.
Second, third result doesn’t look like surprising bacause they are also popular things in S.KOREA.</p>
<h3><a id="62_The_most_frequently_presented_tourism_on_SEOUL_399"></a>6-2. The most frequently presented tourism on SEOUL</h3>
<pre><code>sqlite&gt; select newTable.value, count(*) as num from newTable
   ...&gt; where newTable.key = 'tourism'
   ...&gt; group by newTable.value
   ...&gt; order by num desc;
</code></pre>
<pre><code class="language-sh">motel|<span class="hljs-number">822</span>
hotel|<span class="hljs-number">563</span>
museum|<span class="hljs-number">308</span>
chalet|<span class="hljs-number">277</span>
information|<span class="hljs-number">151</span>
</code></pre>
<p>It is a little akward that the top two are motel and hotel. I don’t know what to say but yes the map proved they are famous.A museum is a kind of inexpensive place in KOREA compared to USA. The cost is like 1~ 5$ for the person to enter the site.
I have no idea what chalet is.</p>
<h3><a id="63_The_most_freqently_presented_city_on_SEOUL_map_414"></a>6-3. The most freqently presented city on SEOUL map</h3>
<pre><code>sqlite&gt; select newTable.value, count(*) as num
   ...&gt; from newTable
   ...&gt; where newTable.key like '%city'
   ...&gt; group by newTable.value
   ...&gt; order by num desc;
</code></pre>
<pre><code class="language-sh">Bucheon|<span class="hljs-number">299</span>
Yongin|<span class="hljs-number">296</span>
Seoul|<span class="hljs-number">279</span>
연수구|<span class="hljs-number">185</span>
안양시 동안구|<span class="hljs-number">142</span>
</code></pre>
<p>The result is kind of surprising that seoul city is just top three. top one and two are just city near seoul even they are not that popular.</p>
<h3><a id="64_The_most_contributed_user_on_SEOUL_map_430"></a>6-4. The most contributed user on SEOUL map</h3>
<pre><code class="language-sh">    create table userTable(chnageset, uid, timestamp, lon, version, user, lat, id);
    insert into userTable(chnageset, uid, timestamp, lon, version, user, lat, id)
    select changeset, uid, timestamp, lon, version, user, lat, id from nodeGroup;
    insert into userTable(chnageset, uid, timestamp, version, user, id)
    select changeset, uid, timestamp, version, user, id from wayGroup;

    select user, count(*) as num from userTable group by user order by num desc <span class="hljs-built_in">limit</span> <span class="hljs-number">20</span>;
</code></pre>
<p>Here is top five</p>
<pre><code class="language-sh">maphunter36|<span class="hljs-number">149684</span>
전기곰돌|<span class="hljs-number">144133</span>
cyana|<span class="hljs-number">80553</span>
IRTC1015|<span class="hljs-number">80090</span>
panhoong|<span class="hljs-number">54795</span>
</code></pre>
<h1><a id="7_Conclusion_448"></a>7. Conclusion</h1>
<p>As I look through the Seoul city data, there are no consistency and make me confusing sometimes. For example, the newTable which have key and value
have to be divided by category. value meant to be specific name of something and key meant to be categorical name of value something like if value is
Timesqure, then key should be tourist attraction or place something like this way.</p>
<p>This I just took value, key from some last rows</p>
<pre><code class="language-sh">residential|landuse
예정|name
residential|highway
residential|highway
숭인동<span class="hljs-number">1</span>나길|name
</code></pre>
<p>It is a little bit unclarified difference between key and value, residential, high way. some korean characters and name (should be changed to upper category such like street name or something else)
So, I am thinking it will be going to be more easier for users to recognize what each values mean when the map contributors use categorical values thoroughly.
not just like, name or address. change them to what name or what address such like street name or school address.
It could be great for using local data scientist to correct the wrong assigned name or values on SEOUL OSM data and give them a coupon like starbucks
or amusement park when they devoted something on the SEOUL map. Even the franchise restaurant owner could let data scientist to work on their
Restaurant information on the OSM data and give them some advantage when they use the restaurant. It could be good for both owner and the scientist just
for a quick commitment</p>
<p>In userTable, Tag value presented too many confusing language selection such as tag value containing korean, english, japanese, chinese and something else.
specially, I think it would be better just using one or two most recognized languages in Asia, Korean and english
It would be better to change the form in order to make it easy to recognize for both korean and english.
There are still things needed to change (make things easy to be recognized)</p>
<pre><code class="language-sh">bridge:name:en &gt;&gt; bridge
route_ref &gt;&gt; route
name:en &gt;&gt; name
addr:postcode &gt;&gt; postcode
bridge:name &gt;&gt; bridge
name:kr_rm &gt;&gt; name:ko
</code></pre>
<p>Even I changed the name ro, gil to -ro, -gil but
there are still exceptional cases even though they are a few. Also top one and two of the tourism which were hotel and motel were pretty surprising.
I think people needed to update more tourism on the map. there are many places to visit as I said cooperated with local computer science students who
can develop the detailed information and give them a reward.</p>

</body></html>

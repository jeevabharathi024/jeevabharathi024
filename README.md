## Hi there 👋

<!--
**jeevabharathi024/jeevabharathi024** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
import os, base64, io, random, zipfile
from xml.sax.saxutils import escape as esc
from xml.dom import minidom
from PIL import Image

BG0,BG1,CARD,LINE,ACC,ACC2,SOFT,TEXT,MUTED='#060B18','#0B1530','#0F1D3F','#1E3A7A','#2F6BFF','#5B9BFF','#A9C7FF','#EAF2FF','#8DA2C8'
SERIF="Georgia,'Times New Roman',serif"
SANS="'Segoe UI','Helvetica Neue',Arial,sans-serif"
MONO="'SF Mono',Consolas,'Courier New',monospace"
ROOT=os.path.dirname(os.path.abspath(__file__))
OUT=os.path.join(ROOT,'profile-readme'); A=os.path.join(OUT,'assets')
os.makedirs(A,exist_ok=True)

NAME='Jeeva Bharathi V'; EMAIL='jeevabharathic297@gmail.com'
PHOTO='/mnt/user-data/uploads/2336010042.jpg'

CSS=""".fade{animation:fade 1.1s ease-out backwards}@keyframes fade{from{opacity:0}to{opacity:1}}
.card{animation:glowb 5s ease-in-out infinite}@keyframes glowb{0%,100%{stroke:@LINE@;stroke-opacity:.85}50%{stroke:@ACC2@;stroke-opacity:1}}
.pulse{animation:pulse 3s ease-in-out infinite}@keyframes pulse{0%,100%{opacity:.45}50%{opacity:1}}
.float{animation:float 4s ease-in-out infinite}@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-9px)}}
.spin{animation:spin 28s linear infinite;transform-box:fill-box;transform-origin:center}@keyframes spin{to{transform:rotate(360deg)}}
.tw{animation:tw 3s ease-in-out infinite}@keyframes tw{0%,100%{opacity:.08}50%{opacity:.95}}
.shine{transform:translateX(-400px);animation:shine 8s ease-in-out infinite}@keyframes shine{0%{transform:translateX(-300px)}55%,100%{transform:translateX(@W@px)}}
.blink{animation:blink 1s steps(1) infinite}@keyframes blink{50%{opacity:0}}
.line{animation:lineIn .6s ease-out backwards}@keyframes lineIn{from{opacity:0;transform:translateX(-14px)}to{opacity:1;transform:translateX(0)}}
.ecg{stroke-dasharray:90 260;animation:ecg 2.6s linear infinite}@keyframes ecg{from{stroke-dashoffset:350}to{stroke-dashoffset:0}}"""

def css(w): return CSS.replace('@LINE@',LINE).replace('@ACC2@',ACC2).replace('@W@',str(w+300))

def svg(w,h,body,label,defs=''):
    return f'''<svg xmlns="http://www.w3.org/2000/svg" width="{w}" height="{h}" viewBox="0 0 {w} {h}" role="img" aria-label="{esc(label)}">
<title>{esc(label)}</title>
<style>{css(w)}</style>
<defs>
<linearGradient id="bg" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="{BG0}"/><stop offset="1" stop-color="{BG1}"/></linearGradient>
<pattern id="dots" width="24" height="24" patternUnits="userSpaceOnUse"><circle cx="2" cy="2" r="1" fill="{LINE}" opacity=".55"/></pattern>
<clipPath id="clip"><rect width="{w}" height="{h}" rx="22"/></clipPath>
<linearGradient id="shg" x1="0" y1="0" x2="1" y2="0"><stop offset="0" stop-color="{SOFT}" stop-opacity="0"/><stop offset=".5" stop-color="{SOFT}" stop-opacity=".13"/><stop offset="1" stop-color="{SOFT}" stop-opacity="0"/></linearGradient>
<filter id="glow" x="-30%" y="-40%" width="160%" height="180%"><feGaussianBlur stdDeviation="4" result="b"><animate attributeName="stdDeviation" values="2;6;2" dur="3.5s" repeatCount="indefinite"/></feGaussianBlur><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
{defs}
</defs>
<rect width="{w}" height="{h}" rx="22" fill="url(#bg)"/>
<rect width="{w}" height="{h}" rx="22" fill="url(#dots)"/>
<rect x=".5" y=".5" width="{w-1}" height="{h-1}" rx="22" fill="none" stroke="{LINE}" stroke-width="1"/>
<g class="fade">
{body}
</g>
<g clip-path="url(#clip)"><polygon class="shine" points="0,0 90,0 50,{h} -40,{h}" fill="url(#shg)"/></g>
<rect class="pulse" x="1.5" y="1.5" width="{w-3}" height="{h-3}" rx="21" fill="none" stroke="{ACC2}" stroke-width="1.5"/>
</svg>
'''

def t(x,y,s,size,fill,fam=SANS,weight='normal',anchor='start',maxw=None,extra=''):
    if maxw is not None:
        assert len(s)*0.6*size<=maxw,('overflow',s,len(s)*0.6*size,maxw)
    return f'<text x="{x}" y="{y}" font-family="{fam}" font-size="{size}" font-weight="{weight}" fill="{fill}" text-anchor="{anchor}" {extra}>{esc(s)}</text>'

def wrap(s,size,maxw):
    lines=[];cur=''
    for wd in s.split():
        c=(cur+' '+wd).strip()
        if len(c)*0.6*size<=maxw: cur=c
        else: lines.append(cur);cur=wd
    if cur: lines.append(cur)
    return lines

def card(x,y,w,h,d=0,rx=16):
    return f'<rect class="card" x="{x}" y="{y}" width="{w}" height="{h}" rx="{rx}" fill="{CARD}" stroke="{LINE}" style="animation-delay:{d}s"/>'

def chip(x,y,s,extra=''):
    w=round(len(s)*7.6+26)
    return (f'<rect x="{x}" y="{y}" width="{w}" height="30" rx="15" fill="{CARD}" stroke="{LINE}"/>'
            +t(x+w/2,y+20,s,13,SOFT,SANS,'bold','middle')),w

def twinkles(n,w,h,seed,x0=0,x1=None):
    r=random.Random(seed); x1=x1 or w; o=''
    for i in range(n):
        o+=f'<circle class="tw" cx="{r.randint(x0+20,x1-20)}" cy="{r.randint(20,h-20)}" r="{r.choice([1.2,1.6,2])}" fill="{ACC2}" style="animation-delay:{round(r.uniform(0,3),2)}s"/>'
    return o

def ecg(x0,x1,y):
    d=f'M{x0},{y}';x=x0
    while x+100<=x1:
        d+=' l30,0 l8,-22 l10,40 l8,-28 l10,10 l34,0';x+=100
    d+=f' H{x1}'
    return (f'<path d="{d}" fill="none" stroke="{LINE}" stroke-width="2" opacity=".6"/>'
            f'<path class="ecg" d="{d}" fill="none" stroke="{ACC2}" stroke-width="2.5" stroke-linecap="round" filter="url(#glow)"/>')

def typed_clip(cid,x,y,w,h,delay,typ):
    tot=delay+typ
    return (f'<clipPath id="{cid}"><rect x="{x}" y="{y}" width="{w}" height="{h}">'
            f'<animate attributeName="width" values="0;0;{w}" keyTimes="0;{round(delay/tot,3)};1" dur="{tot}s" begin="0s" fill="freeze"/></rect></clipPath>')

def save(name,s):
    with open(os.path.join(A,name),'w',encoding='utf-8') as f: f.write(s)

# ---------------- HERO ----------------
im=Image.open(PHOTO).convert('RGB')
im=im.crop((8,40,408,440)).resize((460,460),Image.LANCZOS)
for q in (82,78,72):
    buf=io.BytesIO(); im.save(buf,'JPEG',quality=q,optimize=True)
    if buf.tell()<40000: break
b64=base64.b64encode(buf.getvalue()).decode()
print('photo bytes',buf.tell())

def hero():
    w,h=1200,560
    cx,cy=930,280
    nm_w=640; l1='Building responsive web apps'; l2='from React to Node.js.'
    l1w=round(len(l1)*0.6*26)+10; l2w=round(len(l2)*0.6*26)+10
    defs=(typed_clip('tn',50,125,nm_w,90,0.4,1.8)+typed_clip('t1',50,222,l1w+10,30,2.2,1.4)+typed_clip('t2',50,258,l2w+10,30,3.6,1.2)
          +f'<clipPath id="ph"><circle cx="{cx}" cy="{cy}" r="190"/></clipPath>')
    b=twinkles(26,w,h,7)
    b+=f'<circle class="pulse" cx="{cx}" cy="{cy}" r="205" fill="{ACC}" opacity=".18" filter="url(#glow)"/>'
    b+=f'<circle class="spin" cx="{cx}" cy="{cy}" r="216" fill="none" stroke="{ACC2}" stroke-width="2" stroke-dasharray="6 12"/>'
    b+=f'<image href="data:image/jpeg;base64,{b64}" x="{cx-190}" y="{cy-190}" width="380" height="380" clip-path="url(#ph)" preserveAspectRatio="xMidYMid slice"/>'
    b+=f'<circle cx="{cx}" cy="{cy}" r="194" fill="none" stroke="{ACC2}" stroke-width="4" filter="url(#glow)"/>'
    b+=t(60,100,'SOFTWARE DEVELOPER · MERN FULL STACK',16,ACC2,MONO,'bold',maxw=640)
    b+='<g clip-path="url(#tn)">'+t(60,190,NAME,64,TEXT,SERIF,'bold',maxw=nm_w,extra='filter="url(#glow)"')+'</g>'
    b+='<g clip-path="url(#t1)">'+t(60,244,l1,26,SOFT,SERIF,'normal',extra='font-style="italic"')+'</g>'
    b+='<g clip-path="url(#t2)">'+t(60,280,l2,26,SOFT,SERIF,'normal',extra='font-style="italic"')+'</g>'
    cur_x0=60; cur_x1=60+len(l2)*0.6*26+4
    b+=(f'<rect class="blink" x="{cur_x1}" y="259" width="10" height="26" fill="{ACC2}" filter="url(#glow)">'
        f'<animate attributeName="x" values="{cur_x0};{cur_x0};{cur_x1}" keyTimes="0;{round(3.6/4.8,3)};1" dur="4.8s" begin="0s" fill="freeze"/></rect>')
    x=60
    for s in ['React.js','Node.js','Express.js','JavaScript']:
        c,wd=chip(x,318,s); b+=c; x+=wd+12
    b+=t(60,396,'B.E. CSE · Annamalai University · 2023–2027 · CGPA 7.8',17,TEXT,SANS,maxw=640)
    b+=t(60,428,'Cuddalore, India',17,MUTED,SANS,maxw=640)
    for i,(s,px,py) in enumerate([('React.js',1000,70),('Node.js',1010,452),('Express.js',690,470)]):
        c,wd=chip(px,py,s)
        b+=f'<g class="float" style="animation-delay:{i*0.9}s">{c}</g>'
    return svg(w,h,b,f'{NAME} hero banner with portrait and skills',defs)
save('hero.svg',hero())

# ---------------- TYPING ----------------
def typing():
    w,h=1200,96
    phrases=['Aspiring Full-Stack Developer','MERN stack · React.js · Node.js · Express.js','REST APIs · SQL · SQLite',
             'Quick learner · Team player · Clear communicator','Open to entry-level Software Developer roles']
    fs=22; cw=fs*0.6; x0=40+len('jeeva@portfolio ~ $')*cw+14
    b=f'<rect x="24" y="22" width="{w-48}" height="52" rx="12" fill="{CARD}" stroke="{LINE}" class="card"/>'
    for i,c in enumerate(['#ff5f56','#ffbd2e','#27c93f']): pass
    b+=t(40,56,'jeeva@portfolio ~ $',fs,ACC2,MONO,'bold',maxw=300)
    defs='';pts=[];vals=[]
    for i,p in enumerate(phrases):
        W=round(len(p)*cw)+4; assert x0+W<w-60,p
        s=i*0.2; e=(i+1)*0.2
        kt=[s,s+0.07,e-0.04,e]
        defs+=(f'<clipPath id="p{i}"><rect x="{x0:.1f}" y="30" width="{W if i==0 else 0}" height="40">'
               f'<animate attributeName="width" values="0;0;{W};{W};0;0" keyTimes="0;{s:.3f};{kt[1]:.3f};{kt[2]:.3f};{e:.3f};1" dur="18s" repeatCount="indefinite"/></rect></clipPath>')
        b+=f'<g clip-path="url(#p{i})">'+t(f'{x0:.1f}',56,p,fs,TEXT,MONO,'normal',maxw=W+4)+'</g>'
        seq=[(s,x0),(kt[1],x0+W),(kt[2],x0+W),(e,x0)]
        if i>0: seq=seq[1:]
        for k,v in seq: pts.append(k); vals.append(v)
    kts=';'.join(f'{k:.3f}' for k in pts); vs=';'.join(f'{v:.1f}' for v in vals)
    b+=(f'<rect class="blink" x="{x0:.1f}" y="36" width="11" height="26" fill="{ACC2}" filter="url(#glow)">'
        f'<animate attributeName="x" values="{vs}" keyTimes="{kts}" dur="18s" repeatCount="indefinite"/></rect>')
    return svg(w,h,b,'Terminal banner typing rotating phrases about Jeeva Bharathi V',defs)
save('typing.svg',typing())

# ---------------- TITLES ----------------
def title(fn,num,label,head):
    w,h=1200,120
    t_end=50+len(head)*0.6*40
    assert t_end+60<1100
    b=t(50,46,f'{num} — {label}',15,ACC2,MONO,'bold')
    b+=t(50,92,head,40,TEXT,SERIF,'bold',maxw=900,extra='filter="url(#glow)"')
    ly=72
    b+=f'<line x1="{t_end+30:.0f}" y1="{ly}" x2="1118" y2="{ly}" stroke="{ACC2}" stroke-width="2" filter="url(#glow)" class="pulse"/>'
    b+=f'<circle cx="1134" cy="{ly}" r="6" fill="{ACC2}" filter="url(#glow)" class="pulse"/>'
    save(fn,svg(w,h,b,f'Section {num}: {head}'))
title('title-about.svg','01','ABOUT','Code, Curiosity, Craft')
title('title-recognition.svg','02','RECOGNITION','Papers Worth Presenting')
title('title-work.svg','03','SELECTED WORK','Things I Have Built')
title('title-experience.svg','04','EXPERIENCE','Learning By Shipping')
title('title-toolkit.svg','05','TOOLKIT','My Daily Toolkit')
title('title-connect.svg','06','CONNECT','Let Us Build Together')

# ---------------- ABOUT ----------------
def about():
    w,h=1200,400
    bio=['B.E. CSE student at Annamalai University, class of 2027.',
         'I build responsive web apps with React.js on the front end',
         'and Node.js with Express.js on the back end.',
         'At Top Skilled Academy I built REST APIs and full-stack',
         'features end to end, from components to client-server flow.',
         'I am looking for an entry-level Software Developer role.']
    b=twinkles(10,w,h,3,0,850)
    b+=t(50,64,'// ABOUT ME',15,ACC2,MONO,'bold')
    for i,l in enumerate(bio):
        assert len(l)<62
        b+=t(50,112+i*42,l,20,TEXT if i in (0,5) else SOFT,SANS,'normal',maxw=780)
    stats=[('7.8','CGPA · B.E. CSE'),('2','Projects built'),('2','Papers presented'),('85%','Higher secondary')]
    for i,(v,l) in enumerate(stats):
        x=870+(i%2)*150; y=50+(i//2)*164
        b+=card(x,y,136,150,i*0.6)
        b+=t(x+68,y+68,v,44,TEXT,SERIF,'bold','middle',maxw=120,extra='filter="url(#glow)"')
        for j,ln in enumerate(wrap(l,13,112)):
            b+=t(x+68,y+104+j*18,ln,13,MUTED,MONO,'normal','middle',maxw=120)
    return svg(w,h,b,'About Jeeva Bharathi V with four key stats')
save('about-intro.svg',about())

# ---------------- RECOGNITION ----------------
def recog():
    w,h=1200,360
    items=[('01','2026','PAPER PRESENTED','AI Based Cyber Threat Detection using Machine Learning','National Institute of Technology Puducherry'),
           ('02','2025','PAPER PRESENTED','Big Data Analytics for Business Intelligence','St. Joseph’s College of Arts and Science (Autonomous), Cuddalore'),
           ('03','2025','WORKSHOP','Dataprep & Strategy Workshop','Annamalai University, Chidambaram')]
    b=twinkles(8,w,h,5)
    for i,(n,yr,kind,ti,ve) in enumerate(items):
        x=50+i*375; y=34; cw=350; ch=292
        b+=card(x,y,cw,ch,i*0.8)
        b+=t(x+24,y+44,n,30,ACC2,SERIF,'bold',extra='filter="url(#glow)"')
        b+=t(x+cw-24,y+42,yr,15,SOFT,MONO,'bold','end')
        b+=t(x+24,y+76,kind,12,MUTED,MONO,'bold')
        yy=y+112
        for ln in wrap(ti,19,cw-48):
            b+=t(x+24,yy,ln,19,TEXT,SANS,'bold',maxw=cw-48); yy+=26
        yy+=8
        for ln in wrap(ve,13,cw-48):
            b+=t(x+24,yy,ln,13,SOFT,SANS,'normal',maxw=cw-48); yy+=19
        assert yy<y+ch,(yy,y+ch)
    return svg(w,h,b,'Three recognitions: two paper presentations and one workshop')
save('achievement-motif.svg',recog())

# ---------------- PROJECTS ----------------
def project(fn,num,kind,ti,sub,chips,bullets,metrics):
    w,h=1200,360
    b=twinkles(8,w,h,int(num)*11,0,780)
    b+=card(50,30,1100,300,0)
    b+=t(80,70,f'PROJECT {num} · {kind}',14,ACC2,MONO,'bold')
    b+=t(80,116,ti,38,TEXT,SERIF,'bold',maxw=680,extra='filter="url(#glow)"')
    b+=t(80,146,sub,16,SOFT,SERIF,'normal',maxw=680,extra='font-style="italic"')
    x=80
    for s in chips:
        c,wd=chip(x,166,s); b+=c; x+=wd+10
    assert x<780
    for i,l in enumerate(bullets):
        yy=238+i*34
        b+=f'<circle cx="86" cy="{yy-5}" r="4" fill="{ACC2}" class="pulse" style="animation-delay:{i*0.5}s"/>'
        b+=t(104,yy,l,16,TEXT,SANS,'normal',maxw=680)
    for i,(v,l) in enumerate(metrics):
        y=52+i*92
        b+=f'<rect class="card" x="830" y="{y}" width="290" height="80" rx="14" fill="{BG0}" stroke="{LINE}" style="animation-delay:{i*0.7}s"/>'
        b+=t(850,y+36,v,22,TEXT,SERIF,'bold',maxw=250,extra='filter="url(#glow)"')
        b+=t(850,y+60,l,13,MUTED,MONO,'normal',maxw=250)
    save(fn,svg(w,h,b,f'Project {num}: {ti}'))
project('project-1.svg','01','WEB APP','TravelWorld','A responsive travel website with booking and user accounts.',
    ['JavaScript','Node.js','SQLite'],
    ['Destinations, services, booking, testimonials, login, registration.',
     'Interactive forms and client-side logic written in JavaScript.',
     'Node.js backend with application data managed in SQLite.'],
    [('Booking flow','LOGIN + REGISTRATION'),('Node.js + SQLite','BACKEND AND DATA'),('Responsive UI','DESTINATIONS, SERVICES')])
project('project-2.svg','02','WEB APP','Personal Portfolio','A site that showcases projects, skills and professional info.',
    ['Responsive Design','Portfolio'],
    ['Showcases projects, technical skills and professional information.',
     'Structured About, Projects and Contact sections.',
     'Responsive layout with clear navigation and organized content.'],
    [('3 sections','ABOUT, PROJECTS, CONTACT'),('Responsive','RESPONSIVE WEB DESIGN'),('Clear navigation','ORGANIZED CONTENT')])

# ---------------- EXPERIENCE ----------------
def experience():
    w,h=1200,500
    b=twinkles(8,w,h,9)
    b+=card(50,30,1100,440,0)
    for i,c in enumerate([ACC,ACC2,SOFT]): b+=f'<circle cx="{82+i*22}" cy="62" r="6" fill="{c}" opacity=".85"/>'
    b+=t(170,67,'top-skilled-academy — trainee-intern',14,MUTED,MONO)
    b+=t(80,124,'Trainee Intern',34,TEXT,SERIF,'bold',maxw=500,extra='filter="url(#glow)"')
    b+=t(80,154,'Top Skilled Academy',19,SOFT,SANS,'bold')
    c,wd=chip(1120-190,100,'June 2026 – July 2026'); b+=c
    bl=['Built responsive interfaces with HTML5, CSS3, JavaScript and React.js using reusable components.',
        'Created React.js components and interactive features that improved usability and maintainability.',
        'Developed server-side applications and RESTful APIs with Node.js and Express.js.',
        'Integrated front end and back end to deliver end-to-end features and client-server communication.',
        'Practised debugging, testing and optimization across the full stack.']
    for i,l in enumerate(bl):
        y=210+i*42
        b+=f'<g class="line" style="animation-delay:{round(0.45*(i+1),2)}s">'+t(80,y,'>',16,ACC2,MONO,'bold')+t(106,y,l,16,TEXT,SANS,'normal',maxw=1010)+'</g>'
    b+=f'<rect class="blink" x="80" y="{210+5*42-16}" width="10" height="20" fill="{ACC2}" filter="url(#glow)"/>'
    b+=ecg(80,1120,438)
    return svg(w,h,b,'Experience: Trainee Intern at Top Skilled Academy, June to July 2026')
save('experience.svg',experience())

# ---------------- TOOLKIT ----------------
def toolkit():
    w,h=1200,470
    groups=[('Frontend',['HTML5','CSS3','JavaScript','React.js']),('Backend',['Node.js','Express.js','REST APIs']),
            ('Database',['SQL','SQLite']),('Full-Stack',['Full-Stack Development','Debugging','Testing']),
            ('Teamwork',['Communication','Collaboration','Teamwork']),('Growth',['Quick Learner','Adaptability','Continuous Learning'])]
    b=twinkles(10,w,h,13)
    for i,(g,cs) in enumerate(groups):
        x=50+(i%3)*375; y=30+(i//3)*220; cw=350; ch=190
        b+=card(x,y,cw,ch,i*0.45)
        b+=t(x+24,y+40,f'0{i+1}',13,ACC2,MONO,'bold')+t(x+54,y+42,g,19,TEXT,SERIF,'bold',extra='filter="url(#glow)"')
        cx_,cy_=x+24,y+66
        for s in cs:
            wd=round(len(s)*7.6+26)
            if cx_+wd>x+cw-24: cx_=x+24; cy_+=40
            c,wd=chip(cx_,cy_,s); b+=c; cx_+=wd+10
        assert cy_+30<y+ch-10,g
    return svg(w,h,b,'Toolkit: six grouped skill cards')
save('tech-constellation.svg',toolkit())

# ---------------- FOOTER ----------------
def footer():
    w,h=1200,220
    b=twinkles(16,w,h,21)
    b+=t(600,92,NAME,52,TEXT,SERIF,'bold','middle',maxw=700,extra='filter="url(#glow)"')
    b+=t(600,128,'Building the web, one component at a time.',20,SOFT,SERIF,'normal','middle',maxw=700,extra='font-style="italic"')
    b+=t(600,160,EMAIL,16,ACC2,MONO,'bold','middle',maxw=700)
    b+=ecg(100,1100,196)
    return svg(w,h,b,f'{NAME} signature footer')
save('footer-signature.svg',footer())

# ---------------- README ----------------
def img(fn,alt): return f'<img src="assets/{fn}?v=1" alt="{alt}" width="100%"/>'
mail='https://img.shields.io/badge/Email-jeevabharathic297%40gmail.com-2F6BFF?style=for-the-badge&labelColor=060B18'
readme=f'''<div align="center">

{img('hero.svg','Jeeva Bharathi V, aspiring MERN full-stack developer, hero banner with portrait')}

{img('typing.svg','Terminal banner typing roles and skills')}

<a href="mailto:{EMAIL}"><img src="{mail}" alt="Email Jeeva Bharathi V"/></a>

</div>

{img('title-about.svg','Section 01 About')}

{img('about-intro.svg','About Jeeva Bharathi V with CGPA, projects, papers and higher secondary stats')}

{img('title-recognition.svg','Section 02 Recognition')}

{img('achievement-motif.svg','Two paper presentations and one workshop')}

{img('title-work.svg','Section 03 Selected work')}

{img('project-1.svg','TravelWorld travel booking website project card')}

{img('project-2.svg','Personal Portfolio website project card')}

{img('title-experience.svg','Section 04 Experience')}

{img('experience.svg','Trainee Intern at Top Skilled Academy, June to July 2026')}

{img('title-toolkit.svg','Section 05 Toolkit')}

{img('tech-constellation.svg','Grouped skills: frontend, backend, database, full-stack, teamwork, growth')}

{img('title-connect.svg','Section 06 Connect')}

{img('footer-signature.svg','Jeeva Bharathi V signature footer with email')}
'''
open(os.path.join(OUT,'README.md'),'w',encoding='utf-8').write(readme)

# ---------------- CHECKS + ZIP ----------------
import re
for f in sorted(os.listdir(A)):
    minidom.parse(os.path.join(A,f)); 
for fn in re.findall(r'assets/([\w\-]+\.svg)',readme):
    assert os.path.exists(os.path.join(A,fn)),fn
print('all SVGs valid, README refs ok:',len(os.listdir(A)),'files')
zp=os.path.join(ROOT,'profile-readme.zip')
with zipfile.ZipFile(zp,'w',zipfile.ZIP_DEFLATED) as z:
    z.write(os.path.join(OUT,'README.md'),'README.md')
    for f in sorted(os.listdir(A)): z.write(os.path.join(A,f),'assets/'+f)
print('zip',os.path.getsize(zp))


import os
from numpy import *

env = Environment(tools=['pdflatex','pdftex'])
env.Replace(ENV=os.environ)
mtsutil = 'mtsutil'

def tonemap(name,key=.25,burn=0,crop=None,flip=False):
  cmd = '%s tonemap -p %g,%g'%(mtsutil,key,burn)
  if crop is not None:
    cmd += ' -c %d,%d,%d,%d'%tuple(crop)
  tmp = '%s%s.png'%('flip-' if flip else '',name)
  env.Command(tmp,name+'.exr',[cmd+' -o $TARGET $SOURCE'])
  if flip:
    env.Command(name+'.png',tmp,['convert -flop $SOURCE $TARGET'])

def poster(name,key,crop,flip=False,ratio=6.25):
  tonemap('%s'%name,key=key,crop=crop,flip=flip)
  tonemap('%s-merged'%name,key=key,crop=ratio*asarray(crop),flip=flip)

def adaptive(name,command,shape=(8000,6000)):
  mask = name+'-mask.exr'
  smooth = name+'-smooth.exr'
  sharp = name+'-sharp.exr'
  env.Command([smooth,mask],name+'.exr',['./mask -s %d,%d %s.exr'%(shape[0],shape[1],name)])
  if not os.path.exists(sharp):
    env.Command(sharp,mask,[command+' --width %d --height %d --mask %s -o $TARGET'%(shape[0],shape[1],mask)])
  env.Command(name+'-merged.exr',[smooth,sharp,mask],['./merge %s'%name])

def tiled(name,command,shape=(8000,6000),tile=1000,skip=(),done=()):
  shape = asarray(shape)
  tiles = product((shape+tile-1)//tile)
  images = []
  for t in xrange(tiles):
    if t not in skip:
      target = '%s-%d.exr'%(name,t)
      images.append(target)
      if not os.path.exists(target):
        env.Command(target,[],[command+' --width %d --height %d --tile-size %d --tile %d -o $TARGET'%(shape[0],shape[1],tile,t)])
  env.Command(name+'-merged.exr',images,['./stitch -s %d,%d --tile-size %d $SOURCES'%(shape[0],shape[1],tile)])

def pdf(name):
  env.PDF(name+'.pdf',name+'.tex')

poster('dragon-front',key=.3,crop=(235,0,768,960),flip=1)
if 0:
  tonemap('dragon-down',key=.2,crop=(55,45,1133,852),flip=1)
  tonemap('dragon-side',key=.4,crop=(136,112,1000,600),flip=1)
  tonemap('dragon-side',key=.4,crop=(186,112,900,600),flip=1)
adaptive('dragon-front',command='./render-dragon --samples 512 --data poster-dragon-front --color-seed 194514')
pdf('dragon')

tiled('dragon-gold',skip=xrange(10),command='./render-dragon --samples 512 --data poster-dragon-front --multicolor 0 --integrator erpt --depth 20')

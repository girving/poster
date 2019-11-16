import os
from numpy import *

env = Environment(tools=['pdflatex', 'pdftex'])
env.Replace(ENV=os.environ)
mtsutil = 'mtsutil'

def tonemap(name, key=.25, burn=0, crop=None, flip=False, target=None):
  if target is None:
    target = name
  cmd = '%s tonemap -p %g,%g' % (mtsutil, key, burn)
  if crop is not None:
    cmd += ' -c %d,%d,%d,%d' % tuple(crop)
  tmp = '%s%s.png' % ('flip-' if flip else '', target)
  env.Command(tmp, name + '.exr', [cmd + ' -o $TARGET $SOURCE'])
  if flip:
    env.Command(target + '.png', tmp, ['convert -flop $SOURCE $TARGET'])

def poster(name, key, crop, flip=False, ratio=6.25, small_ratio=1, extend=0):
  crop = asarray(crop)
  tonemap('%s' % name, key=key, crop=small_ratio * crop, flip=flip)
  source = target = '%s-merged' % name 
  if extend:
    source = '%s-large' % name
    env.Command(source + '.exr', target + '.exr', ['./extend --extra %d $SOURCE -o $TARGET' % extend])
  tonemap(source, key=key, crop=ratio * crop, flip=flip, target=target)

def adaptive(name, command, shape=(8000, 6000)):
  small = name + '.exr'
  mask = name + '-mask.exr'
  smooth = name + '-smooth.exr'
  sharp = name + '-sharp.exr'
  if not os.path.exists(small):
    env.Command(small, [], [command + ' --width %d --height %d -o $TARGET' % (shape[0]//2, shape[1]//2)])
  env.Command([smooth, mask], small, ['./mask -s %d,%d $SOURCE' % (shape[0], shape[1])])
  tonemap(name + '-mask')
  if not os.path.exists(sharp):
    env.Command(sharp, mask, [command + ' --width %d --height %d --mask %s -o $TARGET' % (shape[0], shape[1], mask)])
  env.Command(name + '-merged.exr', [smooth, sharp, mask], ['./merge %s' % name])

def tiled(name, command, shape=(8000, 6000), tile=1000, skip=(), path=()):
  shape = asarray(shape)
  tiles = product((shape + tile - 1) // tile)
  images = []
  for t in range(tiles):
    if t not in skip:
      target = '%s-%d.exr' % (name, t)
      images.append(target)
      if not os.path.exists(target):
        cmd = command if t not in path else command.replace('erpt', 'path')
        env.Command(target, [], [cmd + ' --width %d --height %d --tile-size %d --tile %d -o $TARGET'
                                 % (shape[0], shape[1], tile, t)])
  env.Command(name + '-merged.exr', images, ['./stitch -s %d,%d --tile-size %d $SOURCES'
                                             % (shape[0], shape[1], tile)])

def pdf(name):
  env.PDF(name + '.pdf', name + '.tex')

poster('dragon-front', key=.3, crop=(235, 0, 768, 960), flip=1, small_ratio=6.25 / 2)
adaptive('dragon-front', command='./render-dragon --samples 512 --data poster-dragon-front --color-seed 194514')
pdf('dragon')

tiled('dragon-gold', path=list(range(10))+[30,36,37,38,42,43,44,46,47],
      command='./render-dragon --samples 512 --data poster-dragon-front --multicolor 0 --integrator erpt --depth 20')
env.Command('shiny.exr', 'dragon-gold-merged.exr', ['./fixup $SOURCE -o $TARGET'])
tonemap('shiny', key=.5, crop=6.25*asarray((235,0,768,960)), flip=1)
pdf('shiny')

poster('koch-side', key=.4, crop=(100,90,1100,880), extend=200)
adaptive('koch-side',
         command='./render-dragon --view koch-side --data poster-koch-side --samples 512 --color-seed 184853')
pdf('koch')

# Retina background friendly koch
env.Command('koch-background-large.png', 'koch-side-merged.png',
            ['./extend --extra 962,963,0,0 --transpose 1 $SOURCE -o $TARGET'])
env.Command('koch-background-retina.png', 'koch-background-large.png', ['convert -resize 2880x1800 $SOURCE $TARGET'])

poster('gosper-front', key=.3, crop=(314,30,1325,1325), ratio=2)
tonemap('gosper-side-merged', key=.3, crop=2*asarray((392,19,1231,1231)))
poster('gosper-other', key=.3, crop=(344,41,1281,1281), ratio=2)
poster('gosper-back', key=.3, crop=(402,39,1329,1329), ratio=2)
for view in 'front back other side'.split():
  data = 'front' if view == 'other' else view
  command = ('./render-dragon --view gosper-%s --samples 512 --data poster-gosper-%s --color-seed 184811'
             % (view, data))
  if view == 'side':
    tiled('gosper-%s' % view, shape=(4000, 3000), command=command)
  else:
    adaptive('gosper-%s' % view, shape=(4000, 3000), command=command)
pdf('gosper')

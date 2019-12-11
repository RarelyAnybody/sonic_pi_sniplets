# sonic_pi_sniplets

Just some Sonic Pi code

################## OSC


pA = play 60, amp: 0, sustain: 10000
pB = play 60, amp: 0, sustain: 10000
pC = play 60, amp: 0, sustain: 10000
pD = play 60, amp: 0, sustain: 10000


live_loop :oscAAmp do
  use_real_time
  f = sync "/osc/1/rotaryA"
  print(f)
  pA.control amp: f[0]
end

live_loop :oscAFreq do
  use_real_time
  f = sync "/osc/1/faderA"
  print(f)
  pA.control note: 60 + 10*f[0]
end

live_loop :oscBAmp do
  use_real_time
  f = sync "/osc/1/rotaryB"
  print(f)
  pB.control amp: f[0]
end

live_loop :oscBFreq do
  use_real_time
  f = sync "/osc/1/faderB"
  print(f)
  pB.control note: 60 + 10*f[0]
end

live_loop :oscCAmp do
  use_real_time
  f = sync "/osc/1/rotaryC"
  print(f)
  pC.control amp: f[0]
end

live_loop :oscCFreq do
  use_real_time
  f = sync "/osc/1/faderC"
  print(f)
  pC.control note: 60 + 10*f[0]
end

live_loop :oscDAmp do
  use_real_time
  f = sync "/osc/1/rotaryD"
  print(f)
  pD.control amp: f[0]
end

live_loop :oscDFreq do
  use_real_time
  f = sync "/osc/1/faderD"
  print(f)
  pA.control note: 60 + 10*f[0]
end


#################

# Unser Notenvorrat
sc = scale(:a0, :major, num_octaves: 5)

use_random_seed 2
start  = 21 # Kammerton a
seqlen = 4
speed = 180

# p(Prime)=1/16
# p(Sekunde)=1/2
# p(Terz)=1/4
# p(Quarte)=1/8
# p(Quinte)=1/16
define :step do |n|
  m = 0
  if rand_i == 1
    m = 1
  elsif rand_i == 1
    m = 2
  elsif rand_i == 1
    m = 3
  elsif rand_i == 1
    m = 4
  end
  if rand_i == 1
    return n+m
  else
    return n-m
  end
end

define :sequence do |start, length|
  s = []
  n = start
  length.times do
    s.push(n)
    n = step(n)
  end
  return s
end

# kein Test, ob überhaupt möglich
define :seqWithTarget do |start, length, target|
  s = sequence(start, length)
  while s[-1] != target
    s = sequence(start, length)
  end
  return s
end

define :playShorted do |x|
  n = 0
  (x.size-2).times do
    play sc[x[n]]
    with_synth :piano do
      play sc[x[n]-7], on: n%2 == 0
      play sc[x[n]-5], on: n%2 == 0
      play sc[x[n]-3], on: n%2 == 0
    end
    n = n + 1
    sleep 1
  end
  play sc[x[-2]], release: 2
  with_synth :piano do
    play chord(sc[x[-2]]-12, :maj)
  end
  sleep 2
end

A = sequence(start, seqlen)
B = sequence(A[-1], seqlen)
C = seqWithTarget(A[-1], seqlen-1, start).push(0)

if false
  print "base note", sc[21]
  print "A", A
  print "B", B
  print "C", C
end

live_loop :test1 do
  use_synth :pluck
  with_bpm speed do
    print "A"
    A.each { |x|
      play sc[x]
      with_synth :piano do
        play chord(sc[x]-12, :maj)
      end
      sleep 1
    }
    print "B(s)"
    playShorted(B)
    print "A"
    A.each { |x|
      play sc[x]
      sleep 1
    }
    print "A shorted"
    playShorted(A)
    print "A"
    A.each { |x|
      play sc[x]
      sleep 1
    }
    print "C"
    playShorted(C)
  end
  
end


#comment do #
live_loop :rhythm do
  rhythm1 = spread(3,16)
  rhythm2 = spread(7,16)
  tick
  with_bpm speed do
    sample :perc_bell, amp: 0.3, rate: 0.5#, on: rhythm1.look
    sample :perc_snap, amp: 0.6, on: rhythm2.look
    sleep 1
  end
end

###################### a sample from Sam A, edited

live_loop :rhythm1 do
  tick
  with_bpm 560 do
    rhythm1 = spread(11,16)
    sample :perc_snap, amp: 0.5, on: rhythm1.look
    sleep 1
  end
end

live_loop :rhythm do
  sync :rhythm1
  tick
  rhythm1 = spread(5,8)
  rhythm2 = spread(3,8)
  with_bpm 560 do
    sample :bd_haus, amp: 1.0, on: rhythm1.look
    sample :drum_bass_soft, on: rhythm2.look
    sleep 1.99
  end
end

use_synth :tb303
live_loop :acid do
  sync :rhythm1
  use_random_seed 5
  with_bpm 70 do
    event = play 40, amp: 0.2,
      release: 2,
      res: 0.5,
      note_slide: 0.05,
      cutoff_slide: 0.05
    16.times do
      control event,
        note: rand(20..60),
        cutoff: rand(40..120)
      sleep 1.999 / 16
    end
  end
end

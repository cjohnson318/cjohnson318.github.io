---
layout: post
title:  "The Little Man Computer"
date:   2024-08-28 00:00:00 -0700
tags: python
---

This is an example implementation of the Little Man Computer, an instructional
model developed in 1965 by Stuart Madnick to teach computer architecture to
students at MIT. This represents the bare boes of how to write and interpret
assembly code. There are ten instructions: ADD, SUB, STO store, LDA load, BRA
branch always, BRZ branch if zero, BRP branch if positive, INP input, OUT
output, and HLT halt.

{% highlight python %}
from typing import List
import re

class LMC:
    def __init__(self):
        self.mailbox = ['000'] * 100
        self.inp = []
        self.out = []
        self.accumulator = 0
        self.pc = 0 # program counter
        self.negative_flag = 0
        self.dat = {}
        self.stack = []

    def tokenize(self, data: str) -> List[str]:
        result = re.split(r'\s', data)
        result = list(filter(lambda x: x != '', result))
        return result

    def load(self, data: str):
        '''Convert tokens into opcodes, then replace labels with addresses.
        '''
        data = self.tokenize(data)
        for i, instruction in enumerate(data):
            opc = instruction[0]
            opr = instruction[1:]
            if instruction == 'DAT':
                label = self.stack.pop()
                self.dat[label] = f'{i:02}'
            elif instruction == 'HLT':
                self.stack.append('000')
            elif instruction == 'INP':
                self.stack.append('901')
            elif instruction == 'OUT':
                self.stack.append('902')
            elif instruction == 'LDA':
                self.stack.append('5')
            elif instruction == 'STA':
                self.stack.append('3')
            elif instruction == 'ADD':
                self.stack.append('1')
            elif instruction == 'SUB':
                self.stack.append('2')
            elif instruction == 'BRZ':
                self.stack.append('7')
            elif instruction == 'BRP':
                self.stack.append('8')
            elif instruction == 'BRA':
                self.stack.append('6')
            else:
                self.stack.append(instruction)
        i = 0
        while self.stack:
            if self.stack[0] in ['3', '5', '1', '2', '7', '8', '6']:
                opc = self.stack.pop(0)
                if self.stack[0] in self.dat:
                    label = self.stack.pop(0)
                    opr = self.dat[label]
                    
                else:
                    opr = self.stack.pop(0)
                instruction = opc + opr
                self.mailbox[i] = instruction
            else:
                instruction = self.stack.pop(0)
                self.mailbox[i] = instruction
            i += 1
    
    def run(self):
        '''Implement fetch, execute, increment loop.
        '''
        while True:
            # check mailbox
            xx = self.pc
            # fetch pgm instruction
            instruction = self.mailbox[xx]
            # incr PC
            self.pc += 1
            # decode instruction
            opc = instruction[0]
            opr = instruction[1:]
            # fetch data and execute and branch or store
            if opc == '0': # HLT
                break
            elif instruction == '901': # INP
                if self.inp:
                    self.accumulator = int(self.inp.pop(0))
                else:
                    raise RuntimeError('Missing input')
            elif instruction == '902': # OUT
                self.out.append(str(self.accumulator))
            elif opc == '5': # LDA
                self.accumulator = int(self.mbx[int(opr)])
            elif opc == '3': # STA
                self.mailbox[int(opr)] = str(self.accumulator)
            elif opc == '1': # ADD
                self.accumulator += int(self.mailbox[int(opr)])
            elif opc == '2': # SUB
                self.accumulator -= int(self.mailbox[int(opr)])
            elif opc == '7': # BRZ
                if self.accumulator == 0:
                    self.pc = opr
            elif opc == '8': # BRP
                if self.accumulator == 0 or self.negative_flag == 0:
                    self.pc = opr
            elif opc == '6': # BRA
                self.pc = int(opr)
            if self.accumulator < 0:
                self.negative_flag = 1
            else:
                self.negative_flag = 0

program = '''
  INP
  STA A
  INP
  STA B
  ADD A
  OUT
  HLT
A DAT
B DAT
'''
lmc = LMC()
lmc.inp.append('45')
lmc.inp.append('55')
lmc.load(program)
lmc.run()
assert lmc.out == ['100']
{% endhighlight %}


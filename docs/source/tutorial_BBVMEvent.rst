.. currentmodule:: pyqbdi

BasicBlock VMEvent
==================

Introduction
------------

The :ref:`Instrument Callback <api_desc_InstCallback>` allows to insert callback on all or specific instruction.
With a callback at each instruction, it's trivial to follow the execution pointer and obtaint a trace of the execution.
However, the performances are slow, as the execution should be stop at each instruction to execute the callback.
For some traces, a higher level callback can have better perfomances.

The :ref:`api_desc_VMCallback` is called when some conditions are reach during the execution. This tutorial
intoduces 3 `VMEvent`:

- ``BASIC_BLOCK_NEW`` : this event is triggered when a new BasicBlock has been instrumented and add to the cache. It can be used to create a
  coverage of the execution.
- ``BASIC_BLOCK_ENTRY``: this event is triggered before the execution of a basicBlock.
- ``BASIC_BLOCK_EXIT``: this event is triggered after the execution of a basicBlock.

A Basic Block in QBDI
---------------------

QBDI doesn't analyse the whole programm before run it. BasicBlocks are detected dynamically and may not match the BasicBlock given by others tools.
In QBDI, a basicBlock is a sequence of consecutive instructions that doesn't change the instruction pointer except the last one.
Any instruction that may change the instruction pointer (method call, jump, conditionnal jump, method return, ...) are always the
last instruction of a BasicBlock.

QBDI detects the begin of a BasicBlock:

- when the instruction is the first to be execute in QBDI;
- after the end of the previous BasicBlock;
- potentially if the user changes the callback during the execution (add a new callback, clear the cache, return ``BREAK_TO_VM``, ...)

Due to the dynamic detection of the basicBlock, some basicBlock can overlaps others.
For the followed assembly, QBDI can detect 4 differents basicBlock. If the first jump is taken:

    - The begin of the method, between 0x10e9 and 0x1103;
    - The block between 0x1103 and 0x1109;
    - The L2 block between 0x1115 and 0x111f.

If the first jump isn't taken:

    - The begin of the method, between 0x10e9 and 0x1103;
    - The L1+L2 blocks between 0x1109 and 0x111f.

.. code:: asm

    // foo:
    // 10e9:
    push   rbp
    mov    rbp,rsp
    mov    DWORD PTR [rbp-0x14],edi
    mov    edx,DWORD PTR [rbp-0x14]
    mov    eax,edx
    shl    eax,0x2
    add    eax,edx
    mov    DWORD PTR [rbp-0x4],eax
    cmp    DWORD PTR [rbp-0x14],0xa
    jle    1109 <L1>
    // 1103:
    add    DWORD PTR [rbp-0x4],0x33
    jmp    1115 <L2>
    // 1109: L1:
    mov    eax,DWORD PTR [rbp-0x4]
    imul   eax,eax
    add    eax,0x57
    mov    DWORD PTR [rbp-0x4],eax
    // 1115: L2:
    mov    edx,DWORD PTR [rbp-0x4]
    mov    eax,DWORD PTR [rbp-0x14]
    add    eax,edx
    pop    rbp
    ret
    // 111f:


Get BasicBlock Information
--------------------------

To receive BasicBlock information, a ``VMCallback`` should be registered to the VM with ``addVMEventCB`` for
one of ``BASIC_BLOCK_*`` events. Once a registered event occurs, the callback is called with a description of the VM (``VMState``).

The address of the current basicBlock can be retrieved with ``VMState.basicBlockStart`` and ``VMState.basicBlockEnd``.

.. note::

    ``BASIC_BLOCK_NEW`` can be triggered with ``BASIC_BLOCK_ENTRY``. A callback registered with the both event will
    only be called once. You can retrieved the events that trigered the callback in ``VMState.event``.

C API
+++++

Reference: :cpp:type:`VMCallback`, :cpp:func:`qbdi_addVMEventCB`, :cpp:struct:`VMState`

.. code:: c

    VMAction vmcbk(VMInstanceRef vm, const VMState* vmState, GPRState* gprState, FPRState* fprState, void* data) {

        printf("start:0x%" PRIRWORD ", end:0x%" PRIRWORD "%s%s%s\n",
            vmState->basicBlockStart,
            vmState->basicBlockEnd,
            (vmState->event & QBDI_BASIC_BLOCK_NEW)? " BASIC_BLOCK_NEW":"",
            (vmState->event & QBDI_BASIC_BLOCK_ENTRY)? " BASIC_BLOCK_ENTRY":"",
            (vmState->event & QBDI_BASIC_BLOCK_EXIT)? " BASIC_BLOCK_EXIT":"");
        return QBDI_CONTINUE;
    }

    qbdi_addVMEventCB(vm, QBDI_BASIC_BLOCK_NEW | QBDI_BASIC_BLOCK_ENTRY | QBDI_BASIC_BLOCK_EXIT, vmcbk, NULL);

C++ API
+++++++

Reference: :cpp:type:`QBDI::VMCallback`, :cpp:func:`QBDI::VM::addVMEventCB`, :cpp:struct:`QBDI::VMState`

.. code:: cpp

    QBDI::VMAction vmcbk(QBDI::VMInstanceRef vm, const QBDI::VMState* vmState, QBDI::GPRState* gprState, QBDI::FPRState* fprState, void* data) {

        printf("start:0x%" PRIRWORD ", end:0x%" PRIRWORD "%s%s%s\n",
            vmState->basicBlockStart,
            vmState->basicBlockEnd,
            (vmState->event & QBDI::BASIC_BLOCK_NEW)? " BASIC_BLOCK_NEW":"",
            (vmState->event & QBDI::BASIC_BLOCK_ENTRY)? " BASIC_BLOCK_ENTRY":"",
            (vmState->event & QBDI::BASIC_BLOCK_EXIT)? " BASIC_BLOCK_EXIT":"");
        return QBDI::CONTINUE;
    }

    vm.addVMEventCB(QBDI::BASIC_BLOCK_NEW | QBDI::BASIC_BLOCK_ENTRY | QBDI::BASIC_BLOCK_EXIT, vmcbk, nullptr);

PyQBDI API
++++++++++

Reference: :py:func:`pyqbdi.VMCallback`, :py:func:`pyqbdi.VM.addVMEventCB`, :py:class:`pyqbdi.VMState`

.. code:: python

    def vmcbk(vm, vmState, gpr, fpr, data):
        # user callback code

        print("start:0x{:x}, end:0x{:x} {}".format(
            vmState.basicBlockStart,
            vmState.basicBlockEnd,
            vmState.event & (pyqbdi.BASIC_BLOCK_NEW | pyqbdi.BASIC_BLOCK_ENTRY | pyqbdi.BASIC_BLOCK_EXIT) ))
        return pyqbdi.CONTINUE

    vm.addVMEventCB(pyqbdi.BASIC_BLOCK_NEW | pyqbdi.BASIC_BLOCK_ENTRY | pyqbdi.BASIC_BLOCK_EXIT, vmcbk, None)

Frida/QBDI API
++++++++++++++

Reference: :js:func:`VMCallback`, :js:func:`QBDI.addVMEventCB`, :js:class:`VMState`

.. code:: js

    var vmcbk = vm.newVMCallback(function(vm, state, gpr, fpr, data) {
        var msg = "start:" + state.basicBlockStart.toString(16) + ", end:0x" + state.basicBlockEnd.toString(16);
        if (state.event & VMEvent.BASIC_BLOCK_NEW) {
            msg = msg + " BASIC_BLOCK_NEW";
        }
        if (state.event & VMEvent.BASIC_BLOCK_ENTRY) {
            msg = msg + " BASIC_BLOCK_ENTRY";
        }
        if (state.event & VMEvent.BASIC_BLOCK_EXIT) {
            msg = msg + " BASIC_BLOCK_EXIT";
        }
        console.log(msg);
        return VMAction.CONTINUE;
    });

    vm.addVMEventCB(VMEvent.BASIC_BLOCK_NEW | VMEvent.BASIC_BLOCK_ENTRY | VMEvent.BASIC_BLOCK_EXIT, vmcbk, null);


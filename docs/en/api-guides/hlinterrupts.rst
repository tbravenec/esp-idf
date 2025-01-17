High-Level Interrupts
=====================

.. toctree::
   :maxdepth: 1

The Xtensa architecture has support for 32 interrupts, divided over 8 levels, plus an assortment of exceptions. On the {IDF_TARGET_NAME}, the interrupt mux allows most interrupt sources to be routed to these interrupts using the :doc:`interrupt allocator <../api-reference/system/intr_alloc>`. Normally, interrupts will be written in C, but ESP-IDF allows high-level interrupts to be written in assembly as well, allowing for very low interrupt latencies.

Interrupt Levels
----------------

.. only:: esp32

    ===== ================= ====================================================
    Level Symbol            Remark
    ===== ================= ====================================================
    1     N/A               Exception and level 0 interrupts. Handled by ESP-IDF
    2-3   N/A               Medium level interrupts. Handled by ESP-IDF
    4     xt_highint4       Free to use :ref:`(See 1) <hlinterrupts-pin-notes>`
    5     xt_highint5       Normally used by ESP-IDF debug logic :ref:`(See 1) <hlinterrupts-pin-notes>`
    NMI   xt_nmi            Free to use
    dbg   xt_debugexception Debug exception. Called on e.g. a BREAK instruction. :ref:`(See 2) <hlinterrupts-pin-notes>`
    ===== ================= ====================================================


    .. _hlinterrupts-pin-notes:

    The following notes give more information about the items in the tables above.

        1. ESP-IDF debug logic can be configured to run on `xt_highint4` or `xt_highint5` in :ref:`CONFIG_ESP_SYSTEM_CHECK_INT_LEVEL`. Bluetooth's interrupt can be configured to run on level 4 by enabling :ref:`CONFIG_BTDM_CTRL_HLI`. If :ref:`CONFIG_BTDM_CTRL_HLI` is enabled, ESP-IDF debug logic must be running on level 5 interrupt.
        2. If :ref:`CONFIG_BTDM_CTRL_HLI` is enabled, `xt_debugexception` is used to fix `live lock issue <https://www.espressif.com/sites/default/files/documentation/eco_and_workarounds_for_bugs_in_esp32_en.pdf>`_ in ESP32 ECO3.

.. only:: not esp32

    ===== ================= ====================================================
    Level Symbol            Remark
    ===== ================= ====================================================
    1     N/A               Exception and level 0 interrupts. Handled by ESP-IDF
    2-3   N/A               Medium level interrupts. Handled by ESP-IDF
    4     xt_highint4       Normally used by ESP-IDF debug logic
    5     xt_highint5       Free to use
    NMI   xt_nmi            Free to use
    dbg   xt_debugexception Debug exception. Called on e.g. a BREAK instruction.
    ===== ================= ====================================================

Using these symbols is done by creating an assembly file (suffix .S) and defining the named symbols, like this::

        .section .iram1,"ax"
        .global     xt_highint5
        .type       xt_highint5,@function
        .align      4
    xt_highint5:
        ... your code here
        rsr     a0, EXCSAVE_5
        rfi     5

For a real-life example, see the :component_file:`esp_system/port/soc/{IDF_TARGET_PATH_NAME}/highint_hdl.S` file; the panic handler interrupt is implemented there.


Notes
-----

 - Do not call C code from a high-level interrupt; because these interrupts still run in critical sections, this can cause crashes.
   (The panic handler interrupt does call normal C code, but this is OK because there is no intention of returning to the normal code
   flow afterwards.)

   .. only:: esp32

       And if :ref:`CONFIG_BTDM_CTRL_HLI` is enabled, it does call normal C code in high-level interrupt, but this is OK becase we add some protection for it.

 - Make sure your assembly code gets linked in. If the interrupt handler symbol is the only symbol the rest of the code uses from this
   file, the linker will take the default ISR instead and not link the assembly file into the final project. To get around this, in the
   assembly file, define a symbol, like this::

            .global ld_include_my_isr_file
       ld_include_my_isr_file:


The symbol is called ``ld_include_my_isr_file`` here but can have any arbitrary name not defined anywhere else.

Then, in the component CMakeLists.txt, add this file as an unresolved symbol to the ld command line arguments::

   target_link_libraries(${COMPONENT_TARGET} "-u ld_include_my_isr_file")

This should cause the linker to always include a file defining ``ld_include_my_isr_file``, causing the ISR to always be linked in.

 - High-level interrupts can be routed and handled using esp_intr_alloc and associated functions. The handler and handler arguments
   to esp_intr_alloc must be NULL, however.

 - In theory, medium priority interrupts could also be handled in this way. For now, ESP-IDF does not support this.


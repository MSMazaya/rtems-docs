Testing Using Renode
--------------------

`Renode <https://renode.io>`_ is one of the simulators supported for testing
BSPs using rtems-test. Currently, two BSPs are supported
for testing via Renode: RISCV Kendryte K210 and SPARC Leon3. To use it,
the host computer needs to have the necessary dependencies installed.

1. Mono
   Renode requires Mono >= 5.20 (Linux, macOS) or .NET >= 4.7 (Windows).

   .. csv-table::
       :delim: |

       **Linux** | Install the ``mono-complete`` package as per the installation instructions for various Linux distributions, which can be found on `the Mono project website <https://www.mono-project.com/download/stable/#download-lin>`_.
       **macOS** | On macOS, the Mono package can be downloaded directly from `the Mono project website <https://download.mono-project.com/archive/mdk-latest-stable.pkg>`_.
       **Windows** | On Windows 7, download and install `.NET Framework 4.7 <https://www.microsoft.com/net/download/dotnet-framework-runtime>`_. Windows 10 ships with .NET by default, so no action is required.

2. Other dependencies (Linux only)
   On Ubuntu 20.04, you can install the remaining dependencies with the following command::

      $ sudo apt-get install policykit-1 libgtk2.0-0 screen uml-utilities gtk-sharp2 libc6-dev gcc python3 python3-pip

   If you are running a different distribution, you will need to install an analogous list of packages using your package manager; note that the package names may differ slightly.

Installing Renode
^^^^^^^^^^^^^^^^^

All the dependencies are installed, you can go install Renode using the RTEMS Source Builder (the RSB).
First, ``cd`` into the place where you put your RSB directory. Then you install Renode as follows::

$ cd source-builder
$ ../source-builder/sb-set-builder --prefix=$YOUR_PREFIX --trace --bset-tar-file renode

Adding New BSP Test Config
^^^^^^^^^^^^^^^^^^^^^^^^^^

The implementation of Renode testing in ``rtems-test`` can be found in the rtems-tools repository
at ``tester/rtems/renode`` folder. This folder contains all the ``resc`` scripts used to configure
BSP testing with Renode. To add a new test configuration, you first need to check if the BSP is
supported by Renode. You can do this by visiting `Renode's page that lists all supported boards 
<https://renode.readthedocs.io/en/latest/introduction/supported-boards.html>`_.

If the board is listed there, you'll likely find an example ``resc`` script for a simple 
testing configuration for the BSP in `Renode's GitHub repository 
<https://github.com/renode/renode/tree/master/scripts/single-node>`_. You can then copy the 
configuration to a new resc file in ``tester/rtems/renode``. The additional configuration that
you need to add for it to work is the following::

  showAnalyzer "uartAnalyzer" uart Antmicro.Renode.Analyzers.LoggingUartAnalyzer
  uartAnalyzer TimestampFormat None

  set report_repeating_line """
  from Antmicro.Renode.Logging import ConsoleBackend 
  ConsoleBackend.Instance.ReportRepeatingLines = True
  """

  set add_hook """
  def my_match(line):
      ok_to_kill_lines = [
          '*** TEST STATE: USER_INPUT',
          '*** TEST STATE: BENCHMARK',
          '*** END OF TEST ',
          '*** FATAL ***'
      ]
      return any(l in line for l in ok_to_kill_lines)

  def my_hook(line):
      print line
      monitor.Parse("q")

  Antmicro.Renode.Hooks.UartHooksExtensions.AddLineHook(monitor.Machine["sysbus.uart"], my_match, my_hook)
  """

  python $add_hook

  python $report_repeating_line 

You need to add the above script configuration just before the ELF is loaded. In many cases,
this is before the ``sysbus LoadELF`` line. The additional configuration is used to terminate the
test according to the configuration of ``rtems-test`` and to map the test output to stdout.

The next step is to delete the ``$bin`` variable definition from the example script. 
This is because the ``$bin`` variable will be supplied via the ``renode.cfg`` file as the test binary.

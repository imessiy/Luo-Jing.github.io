# Verilator命令及含义
```
$ verilator <options> <verilog-file> <cpp-file>
```
其中，`<verilog_file>` 是需要仿真的 Verilog 文件，`<cpp_file>` 是手动编写的 C++ 激励文件。`<options>` 是一些编译选项，这里列出一些常用的编译选项：

- `--cc`：指定将 Verilog 代码转化为 C++ 代码；
- `--exe`：指定生成目标为可执行文件；
- `--build`：直接编译生成目标文件；
- `--trace`：导出波形文件时需要添加此选项；
- `--top-module` <top-module>：指定 Verilog 顶层模块；
- `--Mdir <build-dir>`：指定生成文件的目录；
- `-CFLAGS <c-flags>`：指定一个 GCC 的编译选项；
- `-I <include-path>`：可以指定一个包含路径。
- `--binary`生成一个可执行的仿真文件
- `-Wall`启用更强的代码纠错

# Makefile运行verilator
官方的一个例子
```
run:
	@echo
	@echo "-- Verilator tracing example"

	@echo
	@echo "-- VERILATE ----------------"
	$(VERILATOR) $(VERILATOR_FLAGS) $(VERILATOR_INPUT)

	@echo
	@echo "-- BUILD -------------------"
# To compile, we can either
# 1. Pass --build to Verilator by editing VERILATOR_FLAGS above.
# 2. Or, run the make rules Verilator does:
#	$(MAKE) -j -C obj_dir -f Vtop.mk
# 3. Or, call a submakefile where we can override the rules ourselves:
	$(MAKE) -j -C obj_dir -f ../Makefile_obj

	@echo
	@echo "-- RUN ---------------------"
	@rm -rf logs
	@mkdir -p logs
	obj_dir/Vtop +trace

	@echo
	@echo "-- COVERAGE ----------------"
	@rm -rf logs/annotated
	$(VERILATOR_COVERAGE) --annotate logs/annotated logs/coverage.dat

	@echo
	@echo "-- DONE --------------------"
	@echo "To see waveforms, open vlt_dump.vcd in a waveform viewer"
	@echo
```
- 首先用verilator执行一些操作，比如将RTL代码转化为C/C++文件
- 然后编译，编译有三种方法，如例子中的注释
- 然后运行，运行时可以带参数，比如`+trace`，在顶层模块进行相应的参数验证及操作
- coverage暂时没管
- 查看波形，波形文件位于`logs`目录

# C++激励文件
下面是官方给的例子：
```
// For std::unique_ptr
#include <memory>

// Include common routines
#include <verilated.h>

// Include model header, generated from Verilating "top.v"
#include "Vtop.h"

// Legacy function required only so linking works on Cygwin and MSVC++
double sc_time_stamp() { return 0; }

int main(int argc, char** argv) {
    // This is a more complicated example, please also see the simpler examples/make_hello_c.

    // Prevent unused variable warnings
    if (false && argc && argv) {}

    // Create logs/ directory in case we have traces to put under it
    Verilated::mkdir("logs");

    // Construct a VerilatedContext to hold simulation time, etc.
    // Multiple modules (made later below with Vtop) may share the same
    // context to share time, or modules may have different contexts if
    // they should be independent from each other.

    // Using unique_ptr is similar to
    // "VerilatedContext* contextp = new VerilatedContext" then deleting at end.
    const std::unique_ptr<VerilatedContext> contextp{new VerilatedContext};
    // Do not instead make Vtop as a file-scope static variable, as the
    // "C++ static initialization order fiasco" may cause a crash

    // Set debug level, 0 is off, 9 is highest presently used
    // May be overridden by commandArgs argument parsing
    contextp->debug(0);

    // Randomization reset policy
    // May be overridden by commandArgs argument parsing
    contextp->randReset(2);

    // Verilator must compute traced signals
    contextp->traceEverOn(true);

    // Pass arguments so Verilated code can see them, e.g. $value$plusargs
    // This needs to be called before you create any model
    contextp->commandArgs(argc, argv);

    // Construct the Verilated model, from Vtop.h generated from Verilating "top.v".
    // Using unique_ptr is similar to "Vtop* top = new Vtop" then deleting at end.
    // "TOP" will be the hierarchical name of the module.
    const std::unique_ptr<Vtop> top{new Vtop{contextp.get(), "TOP"}};

    // Set Vtop's input signals
    top->reset_l = !0;
    top->clk = 0;
    top->in_small = 1;
    top->in_quad = 0x1234;
    top->in_wide[0] = 0x11111111;
    top->in_wide[1] = 0x22222222;
    top->in_wide[2] = 0x3;

    // Simulate until $finish
    while (!contextp->gotFinish()) {
        // Historical note, before Verilator 4.200 Verilated::gotFinish()
        // was used above in place of contextp->gotFinish().
        // Most of the contextp-> calls can use Verilated:: calls instead;
        // the Verilated:: versions just assume there's a single context
        // being used (per thread).  It's faster and clearer to use the
        // newer contextp-> versions.

        contextp->timeInc(1);  // 1 timeprecision period passes...
        // Historical note, before Verilator 4.200 a sc_time_stamp()
        // function was required instead of using timeInc.  Once timeInc()
        // is called (with non-zero), the Verilated libraries assume the
        // new API, and sc_time_stamp() will no longer work.

        // Toggle a fast (time/2 period) clock
        top->clk = !top->clk;

        // Toggle control signals on an edge that doesn't correspond
        // to where the controls are sampled; in this example we do
        // this only on a negedge of clk, because we know
        // reset is not sampled there.
        if (!top->clk) {
            if (contextp->time() > 1 && contextp->time() < 10) {
                top->reset_l = !1;  // Assert reset
            } else {
                top->reset_l = !0;  // Deassert reset
            }
            // Assign some other inputs
            top->in_quad += 0x12;
        }

        // Evaluate model
        // (If you have multiple models being simulated in the same
        // timestep then instead of eval(), call eval_step() on each, then
        // eval_end_step() on each. See the manual.)
        top->eval();

        // Read outputs
        VL_PRINTF("[%" PRId64 "] clk=%x rstl=%x iquad=%" PRIx64 " -> oquad=%" PRIx64
                  " owide=%x_%08x_%08x\n",
                  contextp->time(), top->clk, top->reset_l, top->in_quad, top->out_quad,
                  top->out_wide[2], top->out_wide[1], top->out_wide[0]);
    }

    // Final model cleanup
    top->final();

    // Coverage analysis (calling write only after the test is known to pass)
#if VM_COVERAGE
    Verilated::mkdir("logs");
    contextp->coveragep()->write("logs/coverage.dat");
#endif

    // Return good completion status
    // Don't use exit() or destructor won't get called
    return 0;
}
```
2个指针
- contextp，context->time()表示执行了几个时间单位
- top，根据用户编写的top.v生成的，top->`signal`实现激励文件和RTL模块的交互

while循环中每次翻转clk信号，在clk信号下降沿的时候改变输入信号的值。

**需要注意的是，在最开始clk由0变为1时，RTL代码不会认为这是一个上升沿**

# 相关链接
[verilator官方手册](https://verilator.org/guide/latest/examples.html)

[verilator简易使用指南](https://soc.ustc.edu.cn/CECS/lab2/verilator/)
    函数名                 	执行时机	                  回调函数原型	        常用函数
    INS_AddInstrumentFunction	每执行一条新指令	            (INS, void *)	   INS_InsertCall
    TRACE_AddInstrumentFunction	每执行一个新trace（基本块）	 (TRACE, void *)	BBL_InsertCall
    RTN_AddInstrumentFunction	每执行一个新函数	            (RTN, void *)	   RTN_InsertCall
    IMG_AddInstrumentFunction	每加载一个新映像	            (IMG, void *)	
    IMG_AddUnloadFunction	每卸载一个映像
    
    
    粒度	       操作        	解释
    Instruction	
        INS_MemoryOperandSize(INS, UINT32)	操作数对应的内存大小
        INS_MemoryWriteSize(INS)	写操作的内存大小
        INS_MemoryReadSize(INS)	读操作的内存大小
        INS_IsMemoryRead(INS)	指令是否读内存
        INS_IsMemoryWrite(INS)	指令是否写内存
        INS_HasMemoryRead2(INS)	指令是否含有2个内存读操作数
        INS_IsNop/IsCall/IsBranch/IsCall/IsRet(INS)	判断指令类别
        INS_IsStackRead/Write(INS)	判断指令是否读/写栈内容
        INS_Disassemble(INS)	反汇编指令
        INS_IsSyscall/IsSysRet(INS)	是否系统调用/返回指令
        INS_Address(INS)	指令的地址
        INS_Size(INS)	指令的大小
        INS_Prev/Next(INS)	上/下一条指令
        INS_Rtn(INS)	指令所在的函数
        INS_Opcode(INS ins)	指令的操作码
        INS_RewriteMemoryOperand	修改指令的内存操作数
        INS_InsertDirectJump	插入直接跳转指令
        INS_insertIndirectJump	插入间接跳转指令
        INS_Delete	删除指令
        
    BasicBlock(Single entry, single exit)	
        BBL_InsHead	第一条指令
        BBL_InsTail	最后一条指令
        BBL_Prev/Next	前/后一个基本块
        BBL_Address	基本块的地址
        BBL_NumIns	基本块的大小
        
    TRACE(Single entrance, multiple exit)	
        TRACE_BblHead/BblTail	第一个/最后一个基本块
        TRACE_NumBbl/NumIns	基本块/指令的数目
        TRACE_Address	地址
        TRACE_Size	大小
        TRACE_Rtn	所在的函数
        
    Routine	RTN_InsHead/InsTail	第一条/最后一条指令
        RTN_NumIns	指令的数目
        RTN_Address/Size	地址/大小
        RTN_Sec	函数所在的段
        RTN_Prev/Next	前/后一个函数
        RTN_FindByAddress	根据地址查找函数
        RTN_Sym	函数对应的符号
        
    Section	SEC_RtnHead/RtnTail	第一个/最后一个函数
        SEC_Address/Size	地址/大小
        SEC_Prev/Next	前/后一个段
        SEC_Type	段的类型：代码，数据，调试
        SEC_Data	段的数据内容
        SEC_Mapped	是否映射到内存空间
        SEC_Img	所在的Image
        
    Image	
        IMG_SecHead/SecTail	第一个/最后一个段
        IMG_Prev/Next	前/后一个加载的image
        IMG_RegsymHead	第一个正规符号
        IMG_Type	类型：静态，共享，共享库，可重定位对象
        IMG_FindByAddress	查找image
        APP_ImgHead/ImgTail	第一个/最后一个加载的image
        
        
        
    IARG	IARG_INST_PTR	指令地址
        IARG_REG_REFERENCE	寄存器内容
        IARG_MEMORYREAD_EA	内存读的有效地址
        IARG_MEMORYREAD2_EA	内存读的有效地址
        IARG_MEMORYWRITE_EA	内存写的有效地址
        IARG_MEMORYREAD_SIZE	内存读的大小
        IARG_MEMORYWRITE_SIZE	内存写的大小
        IARG_FUNCARG_CALLSITE_VALUE	函数参数值（caller）
        IARG_FUNCARG_ENTRYPOINT_VALUE	函数参数值（callee）
        IARG_RETURN_IP	返回地址（callee）
        IARG_RETURN_REGS	保存分析函数返回值的寄存器
        IARGf_MEMORYOP_EA	内存操作的有效地址

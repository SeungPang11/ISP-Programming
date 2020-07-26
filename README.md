# ISP_Programming_Assignment1

"""
This security layer inadequately handles A/B storage for files in RepyV2.

Note:
    This security layer uses encasementlib.r2py, restrictions.default, repy.py and Python
    Also you need to give it an application to run.
    python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [attack_program].r2py

"""

TYPE = "type"
ARGS = "args"
RETURN = "return"
EXCP = "exceptions"
TARGET = "target"
FUNC = "func"
OBJC = "objc"


class ABFile():
    def __init__(self, filename, create):
        # globals
        mycontext['debug'] = False
        # local (per object) reference to the underlying file
        self.Afn = filename + '.a'
        self.Bfn = filename + '.b'

        # make the files and add 'SE' to the readat file...
        if create:
            # create = true, if files NOT in listfiles()
            if self.Afn not in listfiles():
                self.Afile = openfile(self.Afn, create)
                self.Bfile = openfile(self.Bfn, create)

                # add 'SE' to the readat file
                self.Afile.writeat('SE', 0)

            # create = true, if files IS in listfiles()
            else:
                self.Afile = openfile(self.Afn, false)
                self.Bfile = openfile(self.Bfn, false)
        else:
             self.Afile = openfile(self.Afn, false)
             self.Bfile = openfile(self.Bfn, false)
             
    def writeat(self, data, offset):
        # Write the requested data to the B file using the sandbox's writeat call
        try:
            self.Bfile.writeat(data, offset)

        except(Exception):
            return

    def readat(self, bytes, offset):
        try:
            # Read from the A file using the sandbox's readat...
            return self.Afile.readat(bytes, offset)

        except(Exception):
            return

    #Update valid file to Afile
    def close(self):
        try:
            dataToWrite = self.Bfile.readat(None, 0)

            #if file is valid, dataToWrite to Afile
            if(dataToWrite[0] == 'S' and dataToWrite[-1] == 'E':)
                self.Afile.writeat(dataToWrite, 0)

            self.Afile.close()
            self.Bfile.close()

        except(Exception):
            return

        #get rid of Bfile

def ABopenfile(filename, create):
    return ABFile(filename, create)


# The code here sets up type checking and variable hiding for you.  You
# should not need to change anything below here.
sec_file_def = {"obj-type": ABFile,
                "name": "ABFile",
                "writeat": {"type": "func", "args": (str, (int, long)), "exceptions": Exception,
                            "return": (int, type(None)), "target": ABFile.writeat},
                "readat": {"type": "func", "args": ((int, long, type(None)), (int, long)), "exceptions": Exception,
                           "return": str, "target": ABFile.readat},
                "close": {"type": "func", "args": None, "exceptions": None, "return": (bool, type(None)),
                          "target": ABFile.close}
                }

CHILD_CONTEXT_DEF["ABopenfile"] = {TYPE: OBJC, ARGS: (str, bool), EXCP: Exception, RETURN: sec_file_def,
                                   TARGET: ABopenfile}

# Execute the user code
secure_dispatch_module()

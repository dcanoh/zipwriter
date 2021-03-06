zipwriter
=========

.. code-block::

 author   : Alexey Melnichuk <alexeymelnichuck@gmail.com>
 version  : 0.1.2
 license  : MIT/X11 
 website  : http://moteus.github.io/ZipWriter/

Library for creating ZIP archive for Lua 5.1/5.2/5.3

.. code-block::

  depends:           zlib 0.4.0 struct 1.4-1
  architectures:     x86
  platforms:         windows

Supports
--------

- write to non seekable stream
- utf8 file names in archives (required iconv)
- ZIP64 (does not use stream:seek())

additinal deps:
- iconv (if not found then file names passed as is)
- alien/ffi (on Windos detect system default codepage)
- lunit (only for test)
- [AesFileEncrypt] (https://github.com/moteus/lua-AesFileEncrypt) (optional)

 
Usage
-----

Make simple archive

.. code-block:: lua

  function make_reader(fname)
    local f = assert(io.open(fname, 'rb'))
    local chunk_size = 1024
    local desc = {
      istext   = true,
      isfile   = true,
      isdir    = false,
      mtime    = 1348048902, -- lfs.attributes('modification') 
      exattrib = 32,         -- get from GetFileAttributesA
    }
    return desc, desc.isfile and function()
      local chunk = f:read(chunk_size)
      if chunk then return chunk end
      f:close()
    end
  end

  local ZipWriter = require "ZipWriter"
  ZipStream = ZipWriter.new()
  ZipStream:open_stream( assert(io.open('readme.zip', 'w+b')), true )
  ZipStream:write('README.md', make_reader('README.md'))
  ZipStream:close()


Reading file from FTP and saving archive on FTP

.. code-block:: lua

  local ZipWriter = require "ZipWriter"
  local FTP = require "socket.ftp"

  local ZipStream = ZipWriter.new()

  -- write zip file directly to ftp
  -- lua 5.1 needs coco
  ZipStream:open_writer(ZipWriter.co_writer(function(reader)
    FTP.put{
      -- ftp params ...
      path = 'test.zip';
      src  = reader;
    }
  end))

  -- read from FTP
  FTP.get{
    -- ftp params ...
    path = 'test.txt'
    sink = ZipWriter.sink(ZipStream, 'test.txt', {isfile=true;istext=1})
  }

  ZipStream:close()


Make encrypted archive

.. code-block:: lua

  local ZipWriter  = require"ZipWriter"
  local AesEncrypt = require"ZipWriter.encrypt.aes"

  ZipStream = ZipWriter.new{
    encrypt = AesEncrypt.new('password')
  }

  -- as before
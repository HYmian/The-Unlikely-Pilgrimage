---
title: "Tail详解"
date: 2017-12-26T22:15:00+08:00
tags: [
    "他山之玉",
]
---

tail是[GNU/Coreutils](http://www.gnu.org/software/coreutils/coreutils.html)的一个子工具

可以打印指定文件的最后十行（默认值，可以修改）到标准输出，也可以持续的监听文件，将更新打印到标准输出里

本文基于coreutils-8.28的代码分析tail是如何实现读取最后N行以及持续监听的功能

<!--more-->

## file_lines

读取最后N行的功能在函数`file_lines`里实现，代码如下
```c
/* Print the last N_LINES lines from the end of file FD.
   Go backward through the file, reading 'BUFSIZ' bytes at a time (except
   probably the first), until we hit the start of the file or have
   read NUMBER newlines.
   START_POS is the starting position of the read pointer for the file
   associated with FD (may be nonzero).
   END_POS is the file offset of EOF (one larger than offset of last byte).
   Return true if successful.  */

static bool
file_lines (const char *pretty_filename, int fd, uintmax_t n_lines,
            off_t start_pos, off_t end_pos, uintmax_t *read_pos)
{
  char buffer[BUFSIZ];
  size_t bytes_read;
  off_t pos = end_pos;

  if (n_lines == 0)
    return true;

  /* Set 'bytes_read' to the size of the last, probably partial, buffer;
     0 < 'bytes_read' <= 'BUFSIZ'.  */
  bytes_read = (pos - start_pos) % BUFSIZ;
  if (bytes_read == 0)
    bytes_read = BUFSIZ;
  /* Make 'pos' a multiple of 'BUFSIZ' (0 if the file is short), so that all
     reads will be on block boundaries, which might increase efficiency.  */
  pos -= bytes_read;
  xlseek (fd, pos, SEEK_SET, pretty_filename);
  bytes_read = safe_read (fd, buffer, bytes_read);
  if (bytes_read == SAFE_READ_ERROR)
    {
      error (0, errno, _("error reading %s"), quoteaf (pretty_filename));
      return false;
    }
  *read_pos = pos + bytes_read;

  /* Count the incomplete line on files that don't end with a newline.  */
  if (bytes_read && buffer[bytes_read - 1] != line_end)
    --n_lines;

  do
    {
      /* Scan backward, counting the newlines in this bufferfull.  */

      size_t n = bytes_read;
      while (n)
        {
          char const *nl;
          nl = memrchr (buffer, line_end, n);
          if (nl == NULL)
            break;
          n = nl - buffer;
          if (n_lines-- == 0)
            {
              /* If this newline isn't the last character in the buffer,
                 output the part that is after it.  */
              if (n != bytes_read - 1)
                xwrite_stdout (nl + 1, bytes_read - (n + 1));
              *read_pos += dump_remainder (false, pretty_filename, fd,
                                           end_pos - (pos + bytes_read));
              return true;
            }
        }

      /* Not enough newlines in that bufferfull.  */
      if (pos == start_pos)
        {
          /* Not enough lines in the file; print everything from
             start_pos to the end.  */
          xlseek (fd, start_pos, SEEK_SET, pretty_filename);
          *read_pos = start_pos + dump_remainder (false, pretty_filename, fd,
                                                  end_pos);
          return true;
        }
      pos -= BUFSIZ;
      xlseek (fd, pos, SEEK_SET, pretty_filename);

      bytes_read = safe_read (fd, buffer, BUFSIZ);
      if (bytes_read == SAFE_READ_ERROR)
        {
          error (0, errno, _("error reading %s"), quoteaf (pretty_filename));
          return false;
        }

      *read_pos = pos + bytes_read;
    }
  while (bytes_read > 0);

  return true;
}
```
先解释下几个参数：

* `pretty_filename` - 文件名，打印到标准输出时用
* `fd` - 文件描述符，用来读取文件内容
* `n_lines` - 需要读取的行数
* `start_pos` - 从fd读取内容的起始位置
* `end_pos` - 从fd读取内容的结束位置
* `read_pos` - 在fd读取到的位置，对这个函数逻辑没有影响

主要逻辑在代码的注释中都讲的很清楚了，有几点细节：

5. 判断文件的大小，如果文件的size不是`BUFSIZ`的整数倍，则先把读取余数，使剩余量对齐为`BUFSIZ`的整数倍
5. `memrchr`函数 - 读取`buffer`中的字符，并与`line_end`比较，如果匹配则返回字符所在的位置，但是采用倒序读取，即从`buffer+n`开始读取
5. 第一次读取的`buffer`，判断头尾是不是`line_end`，如果不是的话，行计数要加一
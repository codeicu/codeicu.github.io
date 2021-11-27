---
layout: post
title: javaSocket简单学习
date: 2018-04-18 16:26:25
tags: Java Framework
---
# 没有看源码, 从字面上来看 很好理解
```
        try {
            ServerSocket ss = new ServerSocket(8888);
            Socket s = ss.accept();

            InputStream is = s.getInputStream();

            int msg = is.read();
            System.out.println(msg);
            is.close();
            s.close();
            ss.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
		
		
		try {
            Socket s = new Socket("127.0.0.1",8888);
            OutputStream os = s.getOutputStream();
            os.write(110);
            os.close();
            s.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
```

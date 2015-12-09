# network-LAN--chat-rooms
a simple practical local network chat rooms


// ServerDlg.cpp : implementation file
//

#include "stdafx.h"
#include "ChatServer.h"
#include "ServerDlg.h"
#include "PortDlg.h"
#include "msg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CAboutDlg dialog used for App About

class CAboutDlg : public CDialog
{
public:
	CAboutDlg();

// Dialog Data
	//{{AFX_DATA(CAboutDlg)
	enum { IDD = IDD_ABOUTBOX };
	//}}AFX_DATA

	// ClassWizard generated virtual function overrides
	//{{AFX_VIRTUAL(CAboutDlg)
	protected:
	virtual void DoDataExchange(CDataExchange* pDX);    // DDX/DDV support
	//}}AFX_VIRTUAL

// Implementation
protected:
	//{{AFX_MSG(CAboutDlg)
	//}}AFX_MSG
	DECLARE_MESSAGE_MAP()
};

CAboutDlg::CAboutDlg() : CDialog(CAboutDlg::IDD)
{
	//{{AFX_DATA_INIT(CAboutDlg)
	//}}AFX_DATA_INIT
}

void CAboutDlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CAboutDlg)
	//}}AFX_DATA_MAP
}

BEGIN_MESSAGE_MAP(CAboutDlg, CDialog)
	//{{AFX_MSG_MAP(CAboutDlg)
		// No message handlers
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CServerDlg dialog

CServerDlg::CServerDlg(CWnd* pParent /*=NULL*/)
	: CDialog(CServerDlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CServerDlg)
		// NOTE: the ClassWizard will add member initialization here
	//}}AFX_DATA_INIT
	// Note that LoadIcon does not require a subsequent DestroyIcon in Win32
	m_hIcon = AfxGetApp()->LoadIcon(IDR_MAINFRAME);

	m_pSocket=NULL;
}

void CServerDlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CServerDlg)
	DDX_Control(pDX, IDC_STATIC_MESSAGES, m_StaMessage);
	DDX_Control(pDX, IDC_STATIC_PORT, m_StaPort);
	DDX_Control(pDX, IDC_STATIC_NUMBER, m_StaNumber);
	DDX_Control(pDX, IDC_LIST_MSG, m_ListMsg);
	//}}AFX_DATA_MAP
}

BEGIN_MESSAGE_MAP(CServerDlg, CDialog)
	//{{AFX_MSG_MAP(CServerDlg)
	ON_WM_SYSCOMMAND()
	ON_WM_PAINT()
	ON_WM_QUERYDRAGICON()
	ON_WM_DESTROY()
	ON_BN_CLICKED(IDOK, OnStopServer)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CServerDlg message handlers

BOOL CServerDlg::OnInitDialog()
{
	CDialog::OnInitDialog();

	// Add "About..." menu item to system menu.

	// IDM_ABOUTBOX must be in the system command range.
	ASSERT((IDM_ABOUTBOX & 0xFFF0) == IDM_ABOUTBOX);
	ASSERT(IDM_ABOUTBOX < 0xF000);

	CMenu* pSysMenu = GetSystemMenu(FALSE);
	if (pSysMenu != NULL)
	{
		CString strAboutMenu;
		strAboutMenu.LoadString(IDS_ABOUTBOX);
		if (!strAboutMenu.IsEmpty())
		{
			pSysMenu->AppendMenu(MF_SEPARATOR);
			pSysMenu->AppendMenu(MF_STRING, IDM_ABOUTBOX, strAboutMenu);
		}
	}

	// Set the icon for this dialog.  The framework does this automatically
	//  when the application's main window is not a dialog
	SetIcon(m_hIcon, TRUE);			// Set big icon
	SetIcon(m_hIcon, FALSE);		// Set small icon
	
	// TODO: Add extra initialization here
	CPortDlg dlg;
	//弹出通信端口对话框
	//并获取通信端口号
	if (dlg.DoModal()==IDOK)
	{
		//创建侦听套接字对象
		m_pSocket=new CListeningSocket(this);
		//用获得的通信端口侦听
		if (m_pSocket->Create(dlg.m_nPort))
		{
			CString strTemp;
			//在窗体上显示通信端口号
			strTemp.Format("端口号：%4d",dlg.m_nPort);
			m_StaPort.SetWindowText(strTemp);
			//错误判断
			if (!m_pSocket->Listen())
			{
				AfxMessageBox("侦听失败！");
				CDialog::OnOK();
				return TRUE;
			}
		}
	}
	
	return TRUE;  // return TRUE  unless you set the focus to a control
}

void CServerDlg::OnSysCommand(UINT nID, LPARAM lParam)
{
	if ((nID & 0xFFF0) == IDM_ABOUTBOX)
	{
		CAboutDlg dlgAbout;
		dlgAbout.DoModal();
	}
	else
	{
		CDialog::OnSysCommand(nID, lParam);
	}
}

// If you add a minimize button to your dialog, you will need the code below
//  to draw the icon.  For MFC applications using the document/view model,
//  this is automatically done for you by the framework.

void CServerDlg::OnPaint() 
{
	if (IsIconic())
	{
		CPaintDC dc(this); // device context for painting

		SendMessage(WM_ICONERASEBKGND, (WPARAM) dc.GetSafeHdc(), 0);

		// Center icon in client rectangle
		int cxIcon = GetSystemMetrics(SM_CXICON);
		int cyIcon = GetSystemMetrics(SM_CYICON);
		CRect rect;
		GetClientRect(&rect);
		int x = (rect.Width() - cxIcon + 1) / 2;
		int y = (rect.Height() - cyIcon + 1) / 2;

		// Draw the icon
		dc.DrawIcon(x, y, m_hIcon);
	}
	else
	{
		CDialog::OnPaint();
	}
}

// The system calls this to obtain the cursor to display while the user drags
//  the minimized window.
HCURSOR CServerDlg::OnQueryDragIcon()
{
	return (HCURSOR) m_hIcon;
}

//在CServerDlg类终止运行时进行的后续处理
void CServerDlg::OnDestroy() 
{
	CDialog::OnDestroy();
	
	//删除侦听套接字
	delete m_pSocket;
	m_pSocket=NULL;
	//向消息列表增加信息
	CString strTemp;
	strTemp="服务器终止服务！";
	m_msgList.AddTail(strTemp);
	//对连接列表进行处理
	while (!m_connectionList.IsEmpty())
	{
		//向每一个连接的客户机发送"服务器终止服务！"的消息
		//并逐个删除已建立的连接
		CClientSocket* pSocket
			=(CClientSocket*)m_connectionList.RemoveHead();
		CMsg* pMsg=AssembleMsg(pSocket);
		pMsg->m_bClose=TRUE;
		SendMsg(pSocket,pMsg);
		if (!pSocket->IsAborted())
		{
			pSocket->ShutDown();
			BYTE Buffer[50];
			while (pSocket->Receive(Buffer,50)>0);
			delete pSocket;
		}
	}
	//删除消息列表
	m_msgList.RemoveAll();
}

//对所有的已连接的客户机的消息进行更新
void CServerDlg::UpdateClients()
{
	for (POSITION pos=m_connectionList.GetHeadPosition();pos!=NULL;)
		{
			//获得连接列表的成员
			CClientSocket* pSocket
				=(CClientSocket*)m_connectionList.GetNext(pos);
			//对客户机成员的消息列表进行分析
			CMsg* pMsg=AssembleMsg(pSocket);
			//客户机的消息列表需要进行更新
			if (pMsg!=NULL)
				//发送新的消息列表
				//更新客户机消息列表的内容
				SendMsg(pSocket,pMsg);
		}
}

//接受连接请求
void CServerDlg::OnAccept()
{
	CClientSocket* pSocket=new CClientSocket(this);
	//接受连接请求
	if (m_pSocket->Accept(*pSocket))
	{
		//对连接套接字初始化
		pSocket->Initialize();
		//假如连接列表
		m_connectionList.AddTail(pSocket);
	}
	else
		delete pSocket;
}

//获取客户机的发送消息
void CServerDlg::OnReceive(CClientSocket* pSocket)
{
	do
	{
		//调用ReadMsg()读取消息
		CMsg* pMsg=ReadMsg(pSocket);
		if (pMsg->m_bClose)
		{
			//如果客户机关闭
			//调用函数CloseSocket()将该与该客户机的连接从连接列表中删除
			DeleteSocket(pSocket);
			break;
		}
	}
	while (!pSocket->m_pArchiveIn->IsBufferEmpty());
	UpdateClients();
}

//分析消息列表
CMsg* CServerDlg::AssembleMsg(CClientSocket* pSocket)
{
	static CMsg msg;
	msg.Init();
	//客户机的消息列表无需更新
	if (pSocket->m_nMsgCount>=m_msgList.GetCount())
		return NULL;

	//客户机的消息列表需要更新
	for (POSITION pos=m_msgList.FindIndex(pSocket->m_nMsgCount);pos!=NULL;)
	{
		//将缺少的消息增加到消息列表最后
		CString strTemp=m_msgList.GetNext(pos);
		msg.m_msgList.AddTail(strTemp);
	}
	pSocket->m_nMsgCount=m_msgList.GetCount();
	UpdateInfo();
	return& msg;
}

//读取消息
CMsg* CServerDlg::ReadMsg(CClientSocket* pSocket)
{
	static CMsg msg;
	TRY
	{
		//读取消息
		pSocket->ReceiveMessage(&msg);
		//显示接收的消息
		DisplayMsg(msg.m_strText);
		//更新消息列表
		m_msgList.AddTail(msg.m_strText);
	}
	CATCH(CFileException,e)
	{
		//错误处理
		CString strTemp;
		strTemp="读取信息错误！";
		msg.m_bClose=TRUE;
		pSocket->Abort();
	}
	END_CATCH

	return &msg;
}

//发送消息
void CServerDlg::SendMsg(CClientSocket* pSocket,CMsg* pMsg)
{
	TRY
	{
		//发送消息
		pSocket->SendMessage(pMsg);
	}
	CATCH(CFileException,e)
	{
		//错误处理
		pSocket->Abort();
		CString strTemp;
		strTemp="发送信息错误！";
		DisplayMsg(strTemp);
	}
	END_CATCH
}

//关闭套接字
void CServerDlg::DeleteSocket(CClientSocket* pSocket)
{
	pSocket->Close();
	POSITION pos,temp;

	for (pos=m_connectionList.GetHeadPosition();pos!=NULL;)
	{
		//对于已经关闭的客户机
		//在消息列表中将已经建立的连接删除
		temp=pos;
		CClientSocket* pSock=(CClientSocket*)m_connectionList.GetNext(pos);
		//匹配成功
		if (pSock==pSocket)
		{
			m_connectionList.RemoveAt(temp);
			UpdateInfo();
			break;
		}
	}
	delete pSocket;
}

//在列表框显示消息
void CServerDlg::DisplayMsg(LPCTSTR lpszMessage)
{
	//在列表框中显示消息
	m_ListMsg.AddString(lpszMessage);

	//更新系统信息
	UpdateInfo();
}

//更新连接信息
void CServerDlg::UpdateInfo()
{
	CString strTemp;
	//更新在线人数
	strTemp.Format("在线人数：%d",m_connectionList.GetCount());
	m_StaNumber.SetWindowText(strTemp);

	//更新信息数目
	strTemp.Format("信息数：%d",m_msgList.GetCount());
	m_StaMessage.SetWindowText(strTemp);
}

//按钮的事件处理函数
void CServerDlg::OnStopServer() 
{
	// TODO: Add your control notification handler code here
	//退出系统
	CDialog::OnOK();
}

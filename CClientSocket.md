// ClientSocket.h: interface for the CClientSocket class.
//
//////////////////////////////////////////////////////////////////////

#if !defined(AFX_CLIENTSOCKET_H__INCLUDED_)
#define AFX_CLIENTSOCKET_H__INCLUDED_

class CMsg;
class CServerDlg;

//用于建立连接和传送接收信息
//客户套接字
class CClientSocket : public CSocket
{
	DECLARE_DYNAMIC(CClientSocket);

//Construction
public:
	CClientSocket(CServerDlg* pDlg);

//Attributes
	//消息数目
	int m_nMsgCount;
	//CSocketFile和CArchive对象，用于数据传输
	CSocketFile* m_pFile;
	CArchive* m_pArchiveIn;
	CArchive* m_pArchiveOut;
	//主对话框类指针
	CServerDlg* m_pDlg;
	//当前是否处于放弃发送的状态
	BOOL IsAborted()	{return m_pArchiveOut==NULL;}

//Operations
public:
	//初始化
	void Initialize();
	//放弃传送
	void Abort();
	//发送消息
	void SendMessage(CMsg* pMsg);
	//接收消息
	void ReceiveMessage(CMsg* pMsg);

//Overridable callbacks
protected:
	//OnReceive事件处理函数
	virtual void OnReceive(int nErrorCode);

//Implementation
public:
	virtual ~CClientSocket();

#ifdef _DEBUG
	virtual void AssertValid() const;
	virtual void Dump(CDumpContext& dc) const;
#endif
};

#endif // !defined(AFX_CLIENTSOCKET_H__INCLUDED_)

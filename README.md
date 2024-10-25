import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

import javax.swing.BorderFactory;

import javax.swing.JButton;
import javax.swing.JComboBox;
import javax.swing.JFrame;
import javax.swing.JLabel;

import javax.swing.JPanel;
import javax.swing.JScrollPane;
import javax.swing.JTable;
import javax.swing.JTextField;
import javax.swing.table.DefaultTableModel;

public class FrmDanhMucSach extends JFrame implements ActionListener {

	private static final long serialVersionUID = 1L;
	private JTextField txtMaSach;
	private JTextField txtTuaSach;
	private JTextField txtTacGia;
	private JTextField txtNamXB;
	private JTextField txtNhaXB;
	private JTextField txtSoTrang;
	private JTextField txtDonGia;
	private JTextField txtISBN;
	private JButton btnThem;
	private JButton btnXoa;
	private JButton btnSua;
	private JButton btnLuu;
	private JComboBox<String> cboMaSach;
	private JTable table;
	private JTextField txtMess;
	private JButton btnXoaRong;
	private int selectedRow = -1; // Biến để lưu hàng đã chọn trong bảng
	private File file = new File("src/DocGhi_QlSach/datas.txt"); // Đường dẫn tới file txt

	// private SachTableModel tableModel;
	private DefaultTableModel tableModel;
	private JButton btnLoc;
//	private JComboBox<String> cboTuaSach;

	public FrmDanhMucSach() {
		setTitle("Quản lý sách");
		setSize(900, 600);
		setLocationRelativeTo(null);
		setResizable(false);
		setDefaultCloseOperation(EXIT_ON_CLOSE);
		buildUI();
		loadDataFromFile(); // Tải dữ liệu từ file txt khi khởi động
	}

	private void buildUI() {

		// Pháº§n North
		JPanel pnlNorth;
		add(pnlNorth = new JPanel(), BorderLayout.NORTH);
		pnlNorth.setPreferredSize(new Dimension(0, 180));
		pnlNorth.setBorder(BorderFactory.createTitledBorder("Records Editor"));
		pnlNorth.setLayout(null); // Absolute layout

		JLabel lblMaSach, lblTuaSach, lblTacGia, lblNamXB, lblNhaXB, lblSoTrang, lblDonGia, lblISBN;
		pnlNorth.add(lblMaSach = new JLabel("Mã sach: "));
		pnlNorth.add(lblTuaSach = new JLabel("Tên Sách "));
		pnlNorth.add(lblTacGia = new JLabel("Tác giả: "));
		pnlNorth.add(lblNamXB = new JLabel("Năm Xuất bản: "));
		pnlNorth.add(lblNhaXB = new JLabel("Nhà  xuất bản "));
		pnlNorth.add(lblSoTrang = new JLabel("Số trang: "));
		pnlNorth.add(lblDonGia = new JLabel("Đơn giá: "));
		pnlNorth.add(lblISBN = new JLabel("International Standard Book Number: "));

		pnlNorth.add(txtMaSach = new JTextField());
		pnlNorth.add(txtTuaSach = new JTextField());
		pnlNorth.add(txtTacGia = new JTextField());
		pnlNorth.add(txtNamXB = new JTextField());
		pnlNorth.add(txtNhaXB = new JTextField());
		pnlNorth.add(txtSoTrang = new JTextField());
		pnlNorth.add(txtDonGia = new JTextField());
		pnlNorth.add(txtISBN = new JTextField());

		pnlNorth.add(txtMess = new JTextField());
		txtMess.setEditable(false);
		txtMess.setBorder(null);
		txtMess.setForeground(Color.red);
		txtMess.setFont(new Font("Arial", Font.ITALIC, 12));

		int w1 = 100, w2 = 300, h = 20;
		lblMaSach.setBounds(20, 20, w1, h);
		txtMaSach.setBounds(120, 20, 200, h);

		lblTuaSach.setBounds(20, 45, w1, h);
//		cboTuaSach.setBounds(120, 45, 300, 20);
		txtTuaSach.setBounds(120, 45, w2, h);
		lblTacGia.setBounds(450, 45, w1, h);
		txtTacGia.setBounds(570, 45, w2, h);

		lblNamXB.setBounds(20, 70, w1, h);
		txtNamXB.setBounds(120, 70, w2, h);
		lblNhaXB.setBounds(450, 70, w1, h);
		txtNhaXB.setBounds(570, 70, w2, h);

		lblSoTrang.setBounds(20, 95, w1, h);
		txtSoTrang.setBounds(120, 95, w2, h);
		lblDonGia.setBounds(450, 95, w1, h);
		txtDonGia.setBounds(570, 95, w2, h);

		lblISBN.setBounds(20, 120, 220, h);
		txtISBN.setBounds(240, 120, 180, h);
		txtMess.setBounds(20, 145, 550, 20);

		// Pháº§n Center
		JPanel pnlCenter;
		add(pnlCenter = new JPanel(), BorderLayout.CENTER);
		pnlCenter.add(btnThem = new JButton("Thêm"));
		pnlCenter.add(btnXoaRong = new JButton("Làm rỗng"));
		pnlCenter.add(btnXoa = new JButton("Xóa"));
		pnlCenter.add(btnSua = new JButton("Sửa"));
		pnlCenter.add(btnLuu = new JButton("Lưu"));
		pnlCenter.add(new JLabel("Tìm theo mã sách: "));
		pnlCenter.add(cboMaSach = new JComboBox<String>());
		cboMaSach.setPreferredSize(new Dimension(100, 25));
		pnlCenter.add(btnLoc = new JButton("Lọc theo tựa sách"));

		// Pháº§n South
		JScrollPane scroll;
		String[] headers = "MaSach;TuaSach;TacGia;NamXuatBan;NhaXuatBan;SoTrang;DonGia;ISBN".split(";");

		tableModel = new DefaultTableModel(headers, 0);
		add(scroll = new JScrollPane(table = new JTable(tableModel), JScrollPane.VERTICAL_SCROLLBAR_ALWAYS,
				JScrollPane.HORIZONTAL_SCROLLBAR_AS_NEEDED), BorderLayout.SOUTH);
		scroll.setBorder(BorderFactory.createTitledBorder("Danh sách các cuốn sách"));
		table.setRowHeight(20);
		scroll.setPreferredSize(new Dimension(0, 350));
		// Thêm MouseListener để lấy dữ liệu từ bảng lên khi nhấn vào hàng
		table.addMouseListener(new MouseAdapter() {
			@Override
			public void mouseClicked(MouseEvent e) {
				selectedRow = table.getSelectedRow();
				if (selectedRow != -1) {
					txtMaSach.setText(tableModel.getValueAt(selectedRow, 0).toString());
					txtTuaSach.setText(tableModel.getValueAt(selectedRow, 1).toString());
					txtTacGia.setText(tableModel.getValueAt(selectedRow, 2).toString());
					txtNamXB.setText(tableModel.getValueAt(selectedRow, 3).toString());
					txtNhaXB.setText(tableModel.getValueAt(selectedRow, 4).toString());
					txtSoTrang.setText(tableModel.getValueAt(selectedRow, 5).toString());
					txtDonGia.setText(tableModel.getValueAt(selectedRow, 6).toString());
					txtISBN.setText(tableModel.getValueAt(selectedRow, 7).toString());
					txtMess.setText("Đã chọn sách để sửa!");
				}
			}
		});
		// Thêm ActionListener cho các nút
		cboMaSach.addActionListener(this);
		btnThem.addActionListener(this);
		btnXoa.addActionListener(this);
		btnXoaRong.addActionListener(this);
		btnSua.addActionListener(this);
		btnLuu.addActionListener(this);
		btnLoc.addActionListener(this);
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		Object source = e.getSource();

		if (source.equals(btnThem)) {
			// Kiểm tra các trường hợp rỗng
			if (txtMaSach.getText().trim().isEmpty() || txtTuaSach.getText().trim().isEmpty()
					|| txtTacGia.getText().trim().isEmpty()) {
				txtMess.setText("Các trường mã sách, tựa sách và tác giả không được để trống!");
				return;
			}

			// Kiểm tra mã sách đã tồn tại hay chưa
			String maSach = txtMaSach.getText().trim();
			for (int i = 0; i < tableModel.getRowCount(); i++) {
				if (tableModel.getValueAt(i, 0).equals(maSach)) {
					txtMess.setText("Mã sách đã tồn tại!");
					return;
				}
			}

			// Thêm sách vào bảng và cboMaSach
			String[] rowData = { maSach, txtTuaSach.getText().trim(), txtTacGia.getText().trim(),
					txtNamXB.getText().trim(), txtNhaXB.getText().trim(), txtSoTrang.getText().trim(),
					txtDonGia.getText().trim(), txtISBN.getText().trim() };
			tableModel.addRow(rowData);
			cboMaSach.addItem(maSach);
			txtMess.setText("Thêm thành công!");

		} else if (source.equals(btnXoa)) {
			if (selectedRow != -1) {
				tableModel.removeRow(selectedRow);
				txtMess.setText("Xóa thành công!");
				clearFields();
			} else {
				txtMess.setText("Vui lòng chọn sách để xóa!");
			}
		} else if (source.equals(btnSua)) {
			if (selectedRow != -1) {
				tableModel.setValueAt(txtMaSach.getText().trim(), selectedRow, 0);
				tableModel.setValueAt(txtTuaSach.getText().trim(), selectedRow, 1);
				tableModel.setValueAt(txtTacGia.getText().trim(), selectedRow, 2);
				tableModel.setValueAt(txtNamXB.getText().trim(), selectedRow, 3);
				tableModel.setValueAt(txtNhaXB.getText().trim(), selectedRow, 4);
				tableModel.setValueAt(txtSoTrang.getText().trim(), selectedRow, 5);
				tableModel.setValueAt(txtDonGia.getText().trim(), selectedRow, 6);
				tableModel.setValueAt(txtISBN.getText().trim(), selectedRow, 7);
				txtMess.setText("Sửa thành công!");
			} else {
				txtMess.setText("Vui lòng chọn sách để sửa!");
			}
		} else if (source.equals(btnLuu)) {
			// Lưu dữ liệu từ JTable vào file txt
			saveDataToFile();
		} else if (source.equals(btnXoaRong)) {
			clearFields();
		}
	}

	// Hàm đọc dữ liệu từ file txt và hiển thị lên JTable
	private void loadDataFromFile() {
		if (!file.exists())
			return;
		try (BufferedReader br = new BufferedReader(new FileReader(file))) {
			String line;
			while ((line = br.readLine()) != null) {
				String[] data = line.split(";");
				tableModel.addRow(data);
				cboMaSach.addItem(data[0]);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	// Hàm lưu dữ liệu từ JTable vào file txt
	private void saveDataToFile() {
		try (BufferedWriter bw = new BufferedWriter(new FileWriter(file))) {
			for (int i = 0; i < tableModel.getRowCount(); i++) {
				for (int j = 0; j < tableModel.getColumnCount(); j++) {
					bw.write(tableModel.getValueAt(i, j).toString());
					if (j < tableModel.getColumnCount() - 1) {
						bw.write(";");
					}
				}
				bw.newLine();
			}
			txtMess.setText("Dữ liệu đã được lưu vào file!");
		} catch (IOException e) {
			txtMess.setText("Lỗi khi lưu dữ liệu!");
			e.printStackTrace();
		}
	}

	private void clearFields() {
		txtMaSach.setText("");
		txtTuaSach.setText("");
		txtTacGia.setText("");
		txtNamXB.setText("");
		txtNhaXB.setText("");
		txtSoTrang.setText("");
		txtDonGia.setText("");
		txtISBN.setText("");
		selectedRow = -1; // Reset hàng đã chọn
	}

	public static void main(String[] args) {
		new FrmDanhMucSach().setVisible(true);
	}
}


import javax.swing.SwingUtilities;

public class Starting {
	public static void main(String[] args) {
		SwingUtilities.invokeLater(new Runnable() {
			@Override
			public void run() {
				FrmDanhMucSach frm = new FrmDanhMucSach();
				frm.setVisible(true);
			}
		});
	}
}


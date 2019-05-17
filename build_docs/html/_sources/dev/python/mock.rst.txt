Mock & patching technique
***************************

Pach decorator
---------------

.. code-block:: python

    #actual function to test
    def my_function():
        return random.randint(1,10)


    #Test begin here
    import unittest
    from unittest.mock import patch
    from watever import my_function

    class TestExample(unittest.TestCase):

        @patch('random.randint')
        def my_test(self, mock_random):
            mock_random.return_value=5
            expected_resutl=5

            result = my_function()
            self.assertEquald(actual, expected)


Patch context
--------------

.. code-block:: python

    #actual function to test
    def my_function():
        return random.randint(1,10)


    #Test begin here
    import unittest
    from unittest.mock import patch
    from watever import my_function

    class TestExample(unittest.TestCase):

        @patch('random.randint')
        def my_test(self, mock_random):
            expected_resutl=5

            with patch('random.randint', return_value=5):
                result = my_function()
            self.assertEquald(actual, expected)

